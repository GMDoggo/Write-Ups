`nmap -A -T4 -p- 192.168.232.46`

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           zFTPServer 6.0 build 2011-10-17
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| total 9680
| ----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
| ----------   1 root     root           25 Feb 10  2011 UninstallService.bat
| ----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
| ----------   1 root     root           17 Aug 13  2011 StopService.bat
| ----------   1 root     root           18 Aug 13  2011 StartService.bat
| ----------   1 root     root         8736 Nov 09  2011 Settings.ini
| dr-xr-xr-x   1 root     root          512 May 14 23:31 log
| ----------   1 root     root         2275 Aug 08  2011 LICENSE.htm
| ----------   1 root     root           23 Feb 10  2011 InstallService.bat
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
|_dr-xr-xr-x   1 root     root          512 Aug 02  2024 accounts
242/tcp  open  http          Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
| http-auth: 
| HTTP/1.1 401 Authorization Required\x0D
|_  Basic realm=Qui e nuce nuculeum esse volt, frangit nucem!
|_http-title: 401 Authorization Required
|_http-server-header: Apache/2.2.21 (Win32) PHP/5.3.8
3145/tcp open  zftp-admin    zFTPServer admin
3389/tcp open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=LIVDA
| Not valid before: 2024-08-01T10:50:21
|_Not valid after:  2025-01-31T10:50:21
|_ssl-date: 2025-05-14T16:32:15+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: LIVDA
|   NetBIOS_Domain_Name: LIVDA
|   NetBIOS_Computer_Name: LIVDA
|   DNS_Domain_Name: LIVDA
|   DNS_Computer_Name: LIVDA
|   Product_Version: 6.0.6001
|_  System_Time: 2025-05-14T16:32:10+00:00
```

# Enum
First thing that stands out is anon login lets check some files out or see if we can upload files.

![](Images/Pasted%20image%2020250514123348.png)

Lets see if we can test for any admin accounts with some basic Brute force

```
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 192.168.232.46 ftp
```

We get an authentication

![](Images/Pasted%20image%2020250514124227.png)

lets login with those credentials now.

![](Images/Pasted%20image%2020250514124253.png)

Lets grab these files.

![](Images/Pasted%20image%2020250514124342.png)

We find the user and hash of the user offsec. lets see if we can crack that hash.

Turns out it was an easy password to crack

![](Images/Pasted%20image%2020250514124442.png)

It doesn't really lead anywhere on the webpage. It also cannot use FTP. Lets see if we can upload files via the share and get execution?

![](Images/Pasted%20image%2020250514124625.png)

Quick test shows we can upload files. Since we saw the index.php we can use a php reverse shell.

Upload and then we can navigate to it using the offsec account for execution. 

Used Ivan php reverse shell from [Here](https://www.revshells.com/)and executed it through the browser.

![](Images/Pasted%20image%2020250514130942.png)
`C:\wamp\bin\apache\Apache2.2.21` is root directory of website

![](Images/Pasted%20image%2020250514131212.png)



# Post Exploit

We have a pretty simple exploitation path as we have Server 2008 without any patches installed. Lets find a kernel exploit and see if we can elevate.

```
OS Name:                   Microsoftr Windows Serverr 2008 Standard                                                         
OS Version:                6.0.6001 Service Pack 1 Build 6001                              
OS Manufacturer:           Microsoft Corporation   
```

We can find the POC [here](https://www.exploit-db.com/exploits/40564)or [here](https://github.com/abatchy17/WindowsExploits/tree/master/MS11-046)and we can simply upload via FTP and run it to see if we get escalation. 

![](Images/Pasted%20image%2020250514133057.png)

![](Images/Pasted%20image%2020250514133115.png)

As borat says "Great success" time to finish out the box.

![](Images/Pasted%20image%2020250514133207.png)

