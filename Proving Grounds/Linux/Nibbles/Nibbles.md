# Enumeration
`nmap -A -T4 -p- 192.168.148.47`

```
21/tcp   open   ftp          vsftpd 3.0.3
22/tcp   open   ssh          OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:62:1f:f5:22:de:29:d4:24:96:a7:66:c3:64:b7:10 (RSA)
|   256 c9:15:ff:cd:f3:97:ec:39:13:16:48:38:c5:58:d7:5f (ECDSA)
|_  256 90:7c:a3:44:73:b4:b4:4c:e3:9c:71:d1:87:ba:ca:7b (ED25519)
80/tcp   open   http         Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Enter a title, displayed at the top of the window.
139/tcp  closed netbios-ssn
445/tcp  closed microsoft-ds
5437/tcp open   postgresql   PostgreSQL DB 11.3 - 11.9
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=debian
| Subject Alternative Name: DNS:debian
| Not valid before: 2020-04-27T15:41:47
|_Not valid after:  2030-04-25T15:41:47
```


# SQL

[Postgres Default Creds](https://github.com/netbiosX/Default-Credentials/blob/master/PostgreSQL-Default-Password-List.md) we can login with default credentials `postgres:postgres`

`psql -h 192.168.148.47 -p 5437 -U postgres`

![](Images/Pasted%20image%2020250503125914.png)

We get the version information Postgres 11.7. Lets see if this is vulnerable.

[Postgres Reverse Shell](https://github.com/squid22/PostgreSQL_RCE)

![](Images/Pasted%20image%2020250503130421.png)

![](Images/Pasted%20image%2020250503130429.png)

Upgrade shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

# Post Exploit

Transfer Linpeas over for post exploitation

![](Images/Pasted%20image%2020250503130652.png)

Interesting SUID values for escalation. Find the GTFO bins and exploit the SUID

![](Images/Pasted%20image%2020250503130949.png)

