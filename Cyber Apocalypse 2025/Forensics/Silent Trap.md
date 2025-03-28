## Description

A critical incident has occurred in Tales from Eldoria, trapping thousands of players in the virtual world with no way to log out. The cause has been traced back to Malakar, a mysterious entity that launched a sophisticated attack, taking control of the developers' and system administrators' computers. With key systems compromised, the game is unable to function properly, which is why players remain trapped in Eldoria. Now, you must investigate what happened and find a way to restore the system, freeing yourself from the game before it's too late.

## Skills Required

- Basic IMAP protocol
- Email/Network Forensics
- Reverse dotnet executable

## Skills Learned

- IMAP authentication, IMAP commands
- Examining network traffic to retrieve essential data
- Applying AES decryption methods to uncover attacker activities

# Solutions
### [1/6]. What is the subject of the first email that the victim opened and replied to?

After taking a look at the PCAP file, we mainly see TCP communications between two IP addresses: **192.168.91.173** and **192.168.91.133**. The IP **192.168.91.173** corresponds to the **mail server (mail.korptech.net)**, while **192.168.91.133** is likely associated with the victim's computer.

Following the HTTP stream we can view what I assume to be a member of the support team viewing customer complaints.
![](Images/Pasted%20image%2020250326222826.png)
After that and some further enumeration and using `frame contains "compose"` we find the first email opened and replied to
![](Images/Pasted%20image%2020250326223508.png)

### [2/6]. On what date and time was the suspicious email sent?

Further analyzing the HTTP traffic with the IMAP server, I ended up filtering on all GET requests. This lead me to a packet right before we see a new IP address. In specific this is frame 638.
![](Images/Pasted%20image%2020250327212224.png)
When we open the HTTP Stream, we see that its an email with a suspicious looking zip file containing a multi-extension file named Eldoria_Balance_Issue_Report.pdf.ext. 
![](Images/Pasted%20image%2020250327212318.png)
Using some of the keywords in the file attachment we can start to traceback what time this email was received. If we look back at step one we found the emails within the inbox and some details alongside them. Looking closely we see
```
(\"INBOX\",6,true,\"\");\nthis.set_rowcount(\"Messages 1 to 7 of 7\",\"INBOX\");\nthis.set_message_coltypes([\"threads\",\"subject\",\"status\",\"fromto\",\"date\",\"size\",\"flag\",\"attachment\"],null,\"from\");\nthis.add_message_row(72,{\"subject\":\"Bug Report - In-game Imbalance Issue in Eldoria\",\"fromto\":\"<span class=\\\"adr\\\"><span title=\\\"proplayer@email.com\\\" class=\\\"rcmContactAddress\\\">proplayer@email.com</span></span>\",\"date\":\"Today 15:46\",\"size\":\"13 KB\"}
```
A subject line that matches the respective payload we found. Another way of doing this is just booting up Network Miner and dumping the PCAP. As we see it provides us an easier way to investigate these email chains by allowing us to open them.
![](Images/Pasted%20image%2020250327213341.png)


### [3/6]. What is the MD5 hash of the malware file?

Using NetworkMiner to get easily get files and read emails out of the PCAP file, we were able to the get the respective ZIP file and its archive password. Using the password to unlock the archive we can easily grab the flag for the malware file by doing a `md5sum` 
### [4/6]. What credentials were used to log into the attacker's mailbox?

Similarly, within NetworkMiner it was able to get the respective clear text credentials for the attacker's mailbox
![](Images/Pasted%20image%2020250327213542.png)
We can also load up the malicious binary to start debugging .NET. Here are the credentials hard coded in the malicious binary. I will go further into debugging for the remaining flags.
![](Images/Pasted%20image%2020250327213627.png)

## Analysis of Malicious IMAP-Based Command and Control Binary

For the remaining two flags we had to reverse engineer and analyze a malicious binary that utilizes IMAP (Internet Message Access Protocol) for command and control (C2) communication. The objective is to understand the binary's functionality, identify its C2 mechanisms, and develop a method to decrypt its communication.

The initial analysis was performed using dnSpy, a .NET debugger and assembly editor, to decompile and inspect the binary's source code. The primary classes identified were `Exor` and `Program`, which encapsulate the core functionalities of the malware.
#### Exor 
The `Exor` class contains the `Encrypt` method, which implements a custom encryption algorithm. This algorithm is a variant of the RC4 stream cipher, utilizing a Key Scheduling Algorithm (KSA) and a Pseudo-Random Generation Algorithm (PRGA).

![](Images/Pasted%20image%2020250327214720.png)
This custom encryption is used to obfuscate the communication between the infected host and the C2 server, making it harder to detect and analyze.
#### `Program` Class: Core Malware Functionality
The `Program` class contains the main logic of the malware, including network communication, command execution, and data encryption/decryption.
#### Enumeration and Command Retrieval
The malware establishes an IMAP connection, logs in using predefined credentials, and searches for emails in a specified folder (e.g., "Drafts" or "INBOX.Drafts"). The subject of these emails is used to identify commands intended for the infected host.
![](Images/Pasted%20image%2020250327215128.png)
This behavior is confirmed by network packet captures, which show the IMAP `SEARCH` and `FETCH` commands being used to retrieve emails containing C2 instructions.
![](Images/Pasted%20image%2020250327215221.png)
![](Images/Pasted%20image%2020250327215214.png)

#### Command Output Transmission (`create` Method)
The `create` method is responsible for sending the output of executed commands back to the C2 server. This is achieved by creating a new email using the IMAP `APPEND` command, with the command output encoded and encrypted in the email body.
![](Images/Pasted%20image%2020250327215425.png)
#### Command Execution (`cmd` and `execute` Methods)

The `cmd` method executes commands using the `cmd.exe` command-line interpreter. The `execute` method orchestrates the retrieval and execution of commands received from the IMAP server. It also handles persistence by copying the malware to the startup directory and provides the capability for self-modification of the IMAP credentials.

![](Images/Pasted%20image%2020250327215925.png)
![](Images/Pasted%20image%2020250327215942.png)

Network packet captures confirm the IMAP `APPEND` command being used to send encrypted command outputs back to the C2 server.
![](Images/Pasted%20image%2020250327220107.png)

#### Encryption/Decryption (`xor` Method)
The `xor` method acts as a wrapper for the `Exor.Encrypt` method, using a hardcoded byte array as the encryption key. This key is used to encrypt command outputs before sending them to the C2 server and to decrypt commands received from the server.
![](Images/Pasted%20image%2020250327220451.png)
This hardcoded key is a critical vulnerability, as it allows for the decryption of all C2 communication.

There is two main flows to this, encrypting all of the commands being sent as well as receiving and decrypting the received data.
- The xor function is used after a command is run, and the output is converted to bytes. This byte array is then passed to the encrypt function, along with the hardcoded key, and the returned encrypted byte array is then base 64 encoded, and sent to the imap server.

- When the program receives an encrypted string from the imap server, it is base 64 decoded, then the resulting byte array is passed to the xor function, and the returned byte array is then converted to a string.

#### Decryption Script (Python)
To decrypt the C2 communication, a Python script was developed that implements the RC4 algorithm and uses the hardcoded key from the malware.

```python
import base64

def rc4_decrypt(ciphertext, key):
    """Decrypts ciphertext using the RC4 algorithm."""
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]

    i = 0
    j = 0
    plaintext = bytearray()
    for char in ciphertext:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) % 256]
        plaintext.append(char ^ K)
    return bytes(plaintext)

rc4_key = [
    168, 115, 174, 213, 168, 222, 72, 36, 91, 209, 242, 128, 69, 99, 195, 164,
    238, 182, 67, 92, 7, 121, 164, 86, 121, 10, 93, 4, 140, 111, 248, 44,
    30, 94, 48, 54, 45, 100, 184, 54, 28, 82, 201, 188, 203, 150, 123,
    163, 229, 138, 177, 51, 164, 232, 86, 154, 179, 143, 144, 22, 134,
    12, 40, 243, 55, 2, 73, 103, 99, 243, 236, 119, 9, 120, 247,
    25, 132, 137, 67, 66, 111, 240, 108, 86, 85, 63, 44, 49, 241,
    6, 3, 170, 131, 150, 53, 49, 126, 72, 60, 36, 144, 248, 55,
    10, 241, 208, 163, 217, 49, 154, 206, 227, 25, 99, 18, 144, 134,
    169, 237, 100, 117, 22, 11, 150, 157, 230, 173, 38, 72, 99, 129,
    30, 220, 112, 226, 56, 16, 114, 133, 22, 96, 1, 90, 72, 162,
    38, 143, 186, 35, 142, 128, 234, 196, 239, 134, 178, 205, 229, 121, 225,
    246, 232, 205, 236, 254, 152, 145, 98, 126, 29, 217, 74, 177, 142, 19,
    190, 182, 151, 233, 157, 76, 74, 104, 155, 79, 115, 5, 18, 204,
    65, 254, 204, 118, 71, 92, 33, 58, 112, 206, 151, 103, 179, 24, 164,
    219, 98, 81, 6, 241, 100, 228, 190, 96, 140, 128, 1, 161, 246,
    236, 25, 62, 100, 87, 145, 185, 45, 61, 143, 52, 8, 227,
    32, 233, 37, 183, 101, 89, 24, 125, 203, 227, 9, 146, 156, 208,
    206, 194, 134, 194, 23, 233, 100, 38, 158, 58, 159
]

try:
    with open("email.txt", "r") as f:
        base64_encoded_data = f.read().strip() # read and remove any extra whitespace.

    encrypted_data = base64.b64decode(base64_encoded_data)
    decrypted_data = rc4_decrypt(encrypted_data, rc4_key)

    print(decrypted_data) # print the decrypted bytes.

except FileNotFoundError:
    print("Error: email.txt not found.")
except base64.binascii.Error:
    print("Error: Invalid base64 data in email.txt.")
except Exception as e:
    print(f"An unexpected error occurred: {e}")
```

This script reads the Base64-encoded and encrypted data from an email file, decrypts it using the RC4 algorithm and the hardcoded key, and prints the decrypted output.
## Summary of Malware Functionality

The malware operates as a remote access tool (RAT) using IMAP for C2 communication. Key functionalities include:

1. **Persistence:** Automatic execution on system startup.
2. **Identification:** Unique computer ID generation.
3. **Command and Control:** IMAP-based command retrieval and execution.
4. **Command Execution:** Execution of arbitrary commands using `cmd.exe`.
5. **Output Transmission:** Encrypted command output sent via IMAP.
6. **Self-Modification:** Ability to change IMAP credentials.
7. **Encryption:** Using a variant of RC4 with a hardcoded key.

This analysis provides a comprehensive understanding of the malware's behavior and allows for the decryption of its C2 communication, aiding in incident response and threat intelligence.

After building out the script we can successfully decrypt the messages being sent back to the C2
![](Images/Pasted%20image%2020250327223124.png)

The remaining flags just require you to continue the decryption of the communication stream with the C2
### [5/6]. What is the name of the task scheduled by the attacker?
![](Images/Pasted%20image%2020250327223334.png)
### [6/6]. What is the API key leaked from the highly valuable file discovered by the attacker?

![](Images/Pasted%20image%2020250327223349.png)







