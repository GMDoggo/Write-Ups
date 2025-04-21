locate and connect to the server running on **caddyshack.umbccd.net**

nslookup returns 130.85.62.85.

A quick nmap scan returns that port 70 is running a service known as gopher
![](Images/Pasted%20image%2020250419132824.png)
Connecting to that gets us 
![](Images/Pasted%20image%2020250419132843.png)
`echo "/Flag.txt" | nc 130.85.62.85 70`
returns the flag
`DawgCTF{60ph3r_15_n07_d34d!}`