# About
Following the discovery of the front company and identifying main bank account as the financial hub, we now focus on the human element. Intelligence suggests the ShadowTalk platform, a dark web forum used by Volnaya operatives, is managed by an academic with quantum computing expertise. Your task is to identify the person behind ShadowTalk by analyzing their communications and social media presence, uncovering OPSEC failures that reveal their true identity. For more information about the mission, please download the briefing. Submit findings in the format: HTB{PERSON_TITLE_SURNAME_OCCUPATION} Example: HTB{DR_SMITH_RESEARCHER} Note: The flag uses only uppercase letters, numbers, and underscores.



# Solve

## Important Users
```
DarkBridge - Alexei?
Quantum7 - Mod
CipherNode - US
```
In our brief we get the following tools
```
1.ShadowTalk - A dark web platform used for secure communications between Operation Blackoutparticipants. Focus on posts by administrator "DarkBridge" and technical discussions about theplatform's security architecture. (Username: CipherNode - Password: cipher2025)

2.SecureChat - An encrypted messaging application used by Operation Blackout collaborators. Reviewconversations in the "Quantum Security Collective" and "ShadowTalk Infrastructure" groups toidentify OPSEC failures.

3.Chirper - A social media platform where technical experts discuss quantum computing andcryptography. Evidence suggests the administrator maintains a professional presence here.

4.GlobalScience Research Database - An academic repository containing research papers, authorprofiles, and institutional affiliations that may help identify the target individual.
```

We are also provided a twitter clone, my guess is that we are supposed to find someone having bad opsec and link that back to the twitter clone for personal name.

We are looking into user `DarkBridge` lets see if we can find anything on ShadowTalk

We find a message mentioning a University and named Alexei

![](../Images/Pasted%20image%2020250523102507.png)

On the twitter clone that is one of the users.

![](../Images/Pasted%20image%2020250523102526.png)


We get some additional information

![](../Images/Pasted%20image%2020250523103132.png)

```
HTB{DR_ALEXEI_KUZNETSOV_PROFESSOR} does not work
Professor of Quantum Computing
```

![](../Images/Pasted%20image%2020250523103209.png)

```
HTB{DR_ALEXEI_KUZNETSOV_QUANTUM_RESEARCHER}
```
