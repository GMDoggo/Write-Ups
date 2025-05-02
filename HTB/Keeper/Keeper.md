# About
Keeper is an easy-difficulty Linux machine that features a support ticketing system that uses default credentials. Enumerating the service, we are able to see clear text credentials that lead to SSH access. With `SSH` access, we can gain access to a KeePass database dump file, which we can leverage to retrieve the master password. With access to the `Keepass` database, we can access the root `SSH` keys, which are used to gain a privileged shell on the host. 

# Enumeration
```
nmap -A -T4 -p- 10.129.229.41
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Enum
Upon first look we have to update our hosts file

![](Images/Pasted%20image%2020250502131228.png)

After doing that we can access the resource.

![](Images/Pasted%20image%2020250502131249.png)

Lets do some directory fuzzing first.
```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://tickets.keeper.htb:80/rt/FUZZ -fc 302
```

That didn't provide any new results, looking up default credentials for this service gets us `root:password` which ends up working

![](Images/Pasted%20image%2020250502132008.png)

Further enumeration of the service gets us the user `inorgaard`

![](Images/Pasted%20image%2020250502132140.png)

![](Images/Pasted%20image%2020250502132256.png)

We find a potential password in the form of `lnorgaard:Welcome2023!` testing that across the services we get a successful auth on SSH.

![](Images/Pasted%20image%2020250502132434.png)

# Post Exploitation

Within lnorgaard directory there is a zip file when unzipped is a KeePassDump file. Lets attempt to dump the credentials to try and get escalation.

![](Images/Pasted%20image%2020250502132618.png)

After some googling I found [CVE-2023-32784](https://github.com/vdohney/keepass-password-dumper?tab=readme-ov-file) which allows an attacker to access the database master password from a memory dump which we are provided.

I am going to copy the files over to my attacker machine

![](Images/Pasted%20image%2020250502132950.png)

Ended up utilizing the python implementation the [POC](https://github.com/matro7sh/keepass-dump-masterkey)

![](Images/Pasted%20image%2020250502134422.png)

After some time I was able to find the missing chars to be `ø` which makes the pass `dgrød med fløde` (side note false)  since its still missing characters, putting that into google gets me `rødgrød med fløde` which appears to be right now.

![](Images/Pasted%20image%2020250502135313.png)

Within the network folder we get a Putty SSH-RSA key.

![](Images/Pasted%20image%2020250502140008.png)

We can copy that key to a file

![](Images/Pasted%20image%2020250502140210.png)

Since we cannot use a putty key with normal ssh we will have to convert it. Then we can connect as root

![](Images/Pasted%20image%2020250502140308.png)




