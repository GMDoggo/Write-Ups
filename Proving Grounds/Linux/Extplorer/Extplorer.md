`nmap -A -T4 -p- 192.168.152.16`


```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
```


Navigating to default port 80 gets us a Wordpress setup

![](Images/Pasted%20image%2020250504150606.png)

Which I don't think we are supposed to do anything with. Lets do other enumeration.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.16:80/FUZZ
```

![](Images/Pasted%20image%2020250504150850.png)

We find a webpage other than wordpress. Instantly tried `admin:admin` which gave us an authentication.

Within the app directories we find a config file with user:hash combos

```
array('admin','21232f297a57a5a743894a0e4a801fc3','/var/www/html','http://localhost','1','','7',1),
array('dora','$2a$08$zyiNvVoP/UuSMgO2rKDtLuox.vYj.3hZPVYq3i4oG3/CtgET7CjjS','/var/www/html','http://localhost','1','','0',1),
```

The hash for dora is a `bcrypt [Blowfish 32/64 X3]` hash. Lets see if we can decrypt.

![](Images/Pasted%20image%2020250504151500.png)

```
$2a$08$zyiNvVoP/UuSMgO2rKDtLuox.vYj.3hZPVYq3i4oG3/CtgET7CjjS:doraemon

dora:doraemon
```

SSH requires a key, so lets login to dora. Nothing there.

Logging back into admin I realize we can upload files. So I loaded a php reverse shell.

![](Images/Pasted%20image%2020250504152101.png)

Lets see if we can get execution.

![](Images/Pasted%20image%2020250504152157.png)

![](Images/Pasted%20image%2020250504152206.png)

One we get a shell, I checked the /home directory to see if dora was also here. She was and the password was reused so we could upgrade to the user dora.

![](Images/Pasted%20image%2020250504152404.png)

![](Images/Pasted%20image%2020250504152457.png)


# Post Exploitation

You know what time it is.

`Vulnerable to CVE-2021-3560`

![](Images/Pasted%20image%2020250504152739.png)

We have disk access
`debugfs /dev/mapper/ubuntu--vg-ubuntu--lv`

![](Images/Pasted%20image%2020250504153607.png)

First lets pick the biggest file system. Within root we have ssh, so lets try and read the root ssh and get escalation that way.

![](Images/Pasted%20image%2020250504153801.png)

There was no SSH key, lets see instead if we can read /etc/shadow
```
root:$6$AIWcIr8PEVxEWgv1$3mFpTQAc9Kzp4BGUQ2sPYYFE/dygqhDiv2Yw.XcU.Q8n1YO05.a/4.D/x4ojQAkPnv/v7Qrw7Ici7.hs0sZiC.:19453:0:99999:7:::
```

Lets see if this is a weak credential.

![](Images/Pasted%20image%2020250504154146.png)

Great success.

![](Images/Pasted%20image%2020250504154219.png)

