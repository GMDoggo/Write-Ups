# Enumeration
`nmap -A -T4 -p- 192.168.148.42`

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp    open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.45.244], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
|_http-title: Ph33r
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp   open  smux        Linux SNMP multiplexer
445/tcp   open  netbios-ssn Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
60000/tcp open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
```

UDP

```
PORT    STATE SERVICE    VERSION
137/udp open  netbios-ns Samba nmbd netbios-ns (workgroup: WORKGROUP)
| nbns-interfaces: 
|   hostname: 0XBABE
|   interfaces: 
|_    192.168.148.42

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; U.C. Davis, ECE Dept. Tom SNMPv3 server (public)
| snmp-sysdescr: Linux 0xbabe.local 2.6.8-4-386 #1 Wed Feb 20 06:15:54 UTC 2008 i686
|_  System uptime: 26m30.98s (159098 timeticks)
| snmp-interfaces: 
|   lo
|     IP address: 127.0.0.1  Netmask: 255.0.0.0
|     Type: softwareLoopback  Speed: 10 Mbps
|     Traffic stats: 0.00 Kb sent, 0.00 Kb received
|   eth0
|     IP address: 192.168.148.42  Netmask: 255.255.255.0
|     MAC address: 00:50:56:86:f7:73 (VMware)
|     Type: ethernetCsmacd  Speed: 100 Mbps
|_    Traffic stats: 265.72 Mb sent, 96.01 Mb received
| snmp-info: 
|   enterprise: U.C. Davis, ECE Dept. Tom
|   engineIDFormat: unknown
|   engineIDData: 9e325869f30c7749
|   snmpEngineBoots: 61
|_  snmpEngineTime: 26m31s
```


# 25
Here is a list of users for port 25

```
PORT   STATE SERVICE
25/tcp open  smtp
| smtp-enum-users: 
|   root
|   admin
|   administrator
|   webadmin
|   sysadmin
|   netadmin
|   guest
|   user
|   web
|_  test

```
# tcp 80
`Apache/1.3.33`

When first navigating to the website the only thing on the page is a bunch of binary. We can go ahead and convert that.

![](Images/Pasted%20image%2020250503100441.png)

The page says `ifyoudontpwnmeuran00b`

Lets do some further web enumeration using fuff.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.148.42:80/FUZZ
```

That found two directories
```
index - 200
doc - 403
```

# tcp 22/60000
Authentication via ssh requires password or key. 


Since I was able to make a user list via SMTP lets see if we can do some general brute force to get an auth.

```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://192.168.148.42:60000 -t 4
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://192.168.148.42:22 -t 4
```

# SMB
We have no auth to any of the shares at the current moment

![](Images/Pasted%20image%2020250503100839.png)


# Port 161

`sudo nmap -sU -p 161 --script snmp-brute 192.168.148.42`

```
PORT    STATE SERVICE
161/udp open  snmp
| snmp-brute: 
|_  public - Valid credentials

Nmap done: 1 IP address (1 host up) scanned in 1.87 seconds
```

from SNMPWalk I found the following processes

```
iso.3.6.1.2.1.25.4.2.1.4.3774 = STRING: "/usr/local/sbin/clamd"
iso.3.6.1.2.1.25.4.2.1.4.3777 = STRING: "/usr/local/sbin/clamav-milter"
```

It appears this has a couple of vulnerabilities we can try from searchsploit

![](Images/Pasted%20image%2020250503103829.png)

# Exploit
`Sendmail with clamav-milter < 0.91.2 - Remote Command Execution`

![](Images/Pasted%20image%2020250503103958.png)

Looking at the exploit it doesn't directly open a shell but rather opens 31337 for us to connect to. Which is pretty standard for a bind shell. Upon running and connecting to the host we have a shell.

![](Images/Pasted%20image%2020250503104319.png)


