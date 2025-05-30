# About
Following the identification of a front company for Volnaya Corporation, intelligence indicates they've established a complex financial network to fund Operation Blackout. A specific bank account serves as the central hub for these operations. Your task is to trace the money flow through offshore entities and identify the specific bank account used to finance Volnaya's clandestine activities. For more information about the mission, please download the briefing. Submit findings in the format: HTB{BANK_NAME_ACCT_NUMBER} Example: HTB{GLOBAL_TRUST_BANK_ACCT_5241897} Note: The flag uses only uppercase letters, numbers, and underscores.


# Solve

We have identified the front company as being `globalynx-industrial.com` so we must find the bank related to this.

Using the offshore leaks page we get the partner bank for globalynx being `Swiss Meridian Bank`
![](../Images/Pasted%20image%2020250523101715.png)

Using this information we can start looking into other tools we have to try and find the link.

Format: HTB{BANK_NAME_ACCT_NUMBER}
```
HTB{Swiss_Meridian_Bank_Acct_9276514}
```

![](../Images/Pasted%20image%2020250523101927.png)

