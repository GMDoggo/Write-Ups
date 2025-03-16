## About

**Access** is an "easy" difficulty machine that highlights how devices associated with physical security may themselves be vulnerable. This machine demonstrates how publicly accessible services like FTP and file shares can provide attackers a foothold, allowing for lateral movement and credential harvesting. It teaches techniques for identifying and exploiting saved credentials.

---

## Initial Enumeration

Using `nmap` to scan the target machine:

`nmap -A -T4 -p- 10.129.104.142`

Results:

```
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: MegaCorp
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE

```

Key Findings:

- **FTP** allows anonymous login.
- **Telnet** is running, potentially weak authentication.
- **HTTP** server (IIS 7.5) with potentially risky methods (TRACE).

---

### FTP Enumeration

After logging in as `anonymous` on the FTP server, two directories are found:

- **Engineers**
- **Backups**

Inside these directories are two files:

- `backup.mdb`
- `Access Control.zip`

We download both files for further analysis.

---

### File Enumeration

Upon inspecting the files:

- The **Access Control** archive is password-protected, specifically the `.pst` file contained inside it.

Let's investigate the **backup.mdb** file using `mdbtools` to interact with the Microsoft Access database.

![](Images/Pasted%20image%2020250316152915.png)

By querying the database, we extract data from the `auth_user` table, which reveals some potentially useful credentials.

![](Images/Pasted%20image%2020250316153720.png) ![](Pasted%20image%2020250316154322.png)

We find the following credentials:

![](Images/Pasted%20image%2020250316154333.png)

`Engineer: access4u@security`

Given the username `Engineer` matches the file from the **Engineers** directory, we try this password to unlock the **Access Control.zip** file, which succeeds.

![](Images/Pasted%20image%2020250316154448.png)

The `.pst` file extracted from the zip is not human-readable, so we use the `readpst` tool to convert it to the **mbox** format, which is easier to read. Upon doing this, we uncover additional credentials:
![](Images/Pasted%20image%2020250316154810.png)

`security: 4Cc3ssC0ntr0ller`

---

## Exploitation

With valid credentials in hand, we note that **Telnet** is open from our earlier scan. We attempt to authenticate over Telnet using the `security:4Cc3ssC0ntr0ller` credentials and successfully establish a shell.

![](Images/Pasted%20image%2020250316155219.png)

After gaining access, we quickly grab the user flag:

![](Images/Pasted%20image%2020250316155255.png)

---

## Post Exploitation

### Privilege Escalation via Cached Credentials

Further enumeration of the file system uncovers a **LNK** file on the public desktop.

![](Images/Pasted%20image%2020250316155456.png)

Examining the strings within the file reveals cached **Administrator** credentials. This indicates we can leverage the cached credentials to grab our final flag.

![](Images/Pasted%20image%2020250316155656.png)

To confirm, we run the following command to view cached credentials:

`cmdkey /list`

![](Images/Pasted%20image%2020250316155805.png)

Now that we have verified the presence of cached credentials, we can use `runas` to execute commands as the Administrator and retrieve the root flag:

`runas /user:ACCESS\Administrator /savecred "cmd /c type C:\Users\Administrator\Desktop\root.txt > C:\Users\root.txt"`

![](Images/Pasted%20image%2020250316161302.png)
This grants us the **root.txt** flag without needing full privilege escalation.

---

### Conclusion

This machine demonstrated how an attacker could gain a foothold by exploiting insecure FTP shares and escalate access using cached credentials. Although privilege escalation wasn’t necessary to complete the challenge, it highlighted a common vulnerability in stored credentials that can lead to full system compromise.
