# About
With the shell company, financial hub, and key personnel identified, our focus shifts to locating physical infrastructure vulnerabilities. Intelligence suggests "Facility Alpha" houses critical Operation Blackout components with exploitable security weaknesses. Your task is to analyze facility documentation, satellite imagery, and personnel information to identify a specific security vulnerability that could enable access to the facility. For more information about the mission, please download the briefing. Submit findings in the format: HTB{LOCATION_VULNERABILITY_TYPE} Example: HTB{NORTH_GATE_CAMERA_BLINDSPOT} Note: The flag uses only uppercase letters, numbers, and underscores.


# Facility Alpha
In the building permit we get `Industrial Zone East, Block 7` which should be the area or location of the building.

On the career page we get a big detail from Alex Rivera

![](../Images/Pasted%20image%2020250523111654.png)

But that is not it because a news report caught the incident

![](../Images/Pasted%20image%2020250523111837.png)

Maybe this is a red herring as it was posted a year before.

![](../Images/Pasted%20image%2020250523113340.png)

```
Key Infrastructure
    Main Administrative Building (Central)
    Service Entrance B (Northeast)
    Power Substation (Southwest)
    Communication Tower (Northwest)
```

```
Submit findings in the format: 
HTB{LOCATION_VULNERABILITY_TYPE} 
Example: HTB{NORTH_GATE_CAMERA_BLINDSPOT} Note: The flag uses only uppercase letters, numbers, and underscores.
```

After formatting for a while and talking with support for clarification I got it.

```
HTB{SERVICE_ENTRANCE_B_CAMERA_BLINDSPOT}
```