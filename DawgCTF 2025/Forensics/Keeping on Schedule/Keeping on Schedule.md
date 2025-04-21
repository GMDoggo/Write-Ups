# About
One of our computers on the company network had some malware on it. We think we cleared of the main payload however it came back. Can you check for any signs of persistence? We are able to provide you a copy of the registry, the sooner the better!
For any registry related challenges, make sure to not overwrite you machines used registry as it is a sensitive system
![](Images/Pasted%20image%2020250419140253.png)
# Solve
We are just trying to find anything within scheduled tasks or any signs of persistence based on the name.
Open with a registry viewer. The following directory is where scheduled tasks live
```
SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks\
```
We find the red team malicious scheduled task in registry

![](../Images/Pasted%20image%2020250419143228.png)

The author of the file is our flag

![](../Images/Pasted%20image%2020250419143241.png)

`DawgCTF{Fun_W1th_T4sks}`

