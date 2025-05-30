# About
Task Force Phoenix has intercepted faint signals from Volnaya’s government complex—a stream of HVAC control traffic buzzing over the wire. Buried in the chatter lies a critical piece of intel, the first whisper of Operation Blackout’s plan to turn the smart building’s systems against its masters.

# Solve

Example TCP Stream

![](Images/Pasted%20image%2020250523143556.png)

Within this we have a key value `4854427b556e6c` and what appears to be some encoded values in red.

Which appears to be hex when decoding the first we get
`HTB{Unl` which actually lines up nice lets grab all of the p-prefixed values and try and decrypt them

![](Images/Pasted%20image%2020250523143854.png)

it does not appear fully complete but we can assume it will be disruption} if i had to guess

![](Images/Pasted%20image%2020250523144134.png)

decrypting that final block gets us the remainder of our flag.

```
HTB{Unl0ck1ng_th3_v3nts_0f_d15rupt10n}
```

