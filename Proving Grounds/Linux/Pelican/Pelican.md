`nmap -A -T4 -p- 192.168.148.98`

```
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
|_http-server-header: CUPS/2.2 IPP/2.1
|_http-title: Forbidden - CUPS v2.2.10
| http-methods: 
|_  Potentially risky methods: PUT
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
8080/tcp  open  http        Jetty 1.0
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(1.0)
8081/tcp  open  http        nginx 1.14.2
|_http-title: Did not follow redirect to http://192.168.148.98:8080/exhibitor/v1/ui/index.html
|_http-server-header: nginx/1.14.2
34051/tcp open  java-rmi    Java RMI
```


Found Exploit for Exhibitor Web UI [Here](https://www.exploit-db.com/exploits/48654) which got me a local shell.

![](Images/Pasted%20image%2020250503132204.png)

![](Images/Pasted%20image%2020250503132213.png)

Which gets us the local.txt.

![](Images/Pasted%20image%2020250503132326.png)


# Post Exploitation

Transfer and run linpeas before moving forward.

![](Images/Pasted%20image%2020250503132550.png)

While it doesn't directly lead to an easy escalation it will allow us to read files as root.

![](Images/Pasted%20image%2020250503132855.png)

`sudo gcore 494` followed by `strings core.494` sorting through the dump finds us 

![](Images/Pasted%20image%2020250503133027.png)

`root:ClogKingpinInning731`

![](Images/Pasted%20image%2020250503133100.png)







