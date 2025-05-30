# About
Recent attacks on critical infrastructure across three European cities have been linked to a sophisticated threat actor dubbed ""SVIR"" (Special Volnaya Intelligence Regiment). Intelligence suggests that Volnaya Corporation, a multinational conglomerate with legitimate operations, is secretly orchestrating these attacks through a network of shell companies and contractors. Your task is to identify the shell company being used by Volnaya Corporation to procure and deploy specialized Industrial Control System (ICS) components for their operation. For more information about the mission, please download the briefing. Submit findings in the format: HTB{IDENTIFIED_SHELL_COMPANY_NAME} Example: HTB{TECHNOSOFT_SOLUTIONS_LIMITED} Note: The flag uses only uppercase letters, numbers, and underscores.

# Breaking down the Brief
Tools are the docker containers.

```
http://94.237.123.89:34551 - Domain Info
http://94.237.123.89:56010/ - LeakDocs
http://94.237.123.89:42366/ - CorpVaults
http://94.237.123.89:51514/ - Global Ship DB

```

## Primary Domain
`globalynx-industrial.com`

![](../Images/Pasted%20image%2020250523095441.png)


## Leaked Docs
![](../Images/Pasted%20image%2020250523095744.png)

![](../Images/Pasted%20image%2020250523095756.png)

Deliverable was 
```
DELIVERABLE
Submit your findings in the standard Task Force Phoenix format:
HTB{identified_shell_company_name_front}
Example format: HTB{TechnoSoft_Solutions_Limited_front}
```

Combined with the consistent mentions of Globalynx Industrial Systems gets us the flag.

```
HTB{GLOBALYNX_INDUSTRIAL_SYSTEMS_FRONT}
```