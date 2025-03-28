# Synopsis
EncryptedScroll is a Very Easy reversing challenge. Players will analyze some basic arithmetic to extract the flag from the binary.
## Skills Required

- Familiarity with static and dynamic analysis tools
- Knowledge of simple obfuscation techniques
- Ability to bypass basic anti-debugging mechanisms

## Skills Learned

- How to analyze a compiled binary to extract hardcoded secrets
- Understanding string manipulation and character encoding in compiled programs
# Solution
Upon running the provided binary, we are presented with a prompt, suggesting we need to input a "mage's spell" to proceed.
![](Pasted%20image%2020250327235958.png)
A quick examination of the file type reveals that it is an ELF executable.
![](Pasted%20image%2020250328000024.png)

Time to load up GHIDRA and get to work analyzing the code. (Thank god this was not stripped)

## Analysis
Within GHIDRA, we identify the `main` function and pinpoint the key function calls:
![](Pasted%20image%2020250328000303.png)
- display_scroll(); // Display the scroll message 
- printf(&DAT_00102220); // Display a message Assume its The ancient scroll hums with magical energy. Enter the mage’s spell: 
- isoc99_scanf(&DAT_00102268, local_48); // Read user input (the mage's spell) 
- decrypt_message(local_48); // Decrypt and validate input (edited)

Inspecting the `decrypt_message` function reveals the following crucial operations:
![](Pasted%20image%2020250328000250.png)

* `local_10 = *(long *)(in_FS_OFFSET + 0x28)`: This retrieves the stack canary value, a security measure against buffer overflows. Since this isn't a pwn challenge, it's not our primary concern.
* ``builtin_strncpy(local_38, "IUC|t2nqm4\`gm5h\`5s2uin4u2d~", 0x1c)``: This copies an encrypted string into the `local_38` buffer.
* The following loop decrypts the string:
```c
for (local_3c = 0; local_38[local_3c] != '\0'; local_3c = local_3c + 1) { local_38[local_3c] = local_38[local_3c] + -1; }
```
This loop iterates through the encrypted string, subtracting 1 from the ASCII value of each character. This indicates a ROT-1 cipher.

To decrypt the string, I created a simple Python script:
```python
encrypted_str = "IUC|t2nqm4`gm5h`5s2uin4u2d~"
decrypted_str = ''.join(chr(ord(c) - 1) for c in encrypted_str)
print(decrypted_str)
```

Running this script yields the decrypted flag:
![](Pasted%20image%2020250328001048.png)

