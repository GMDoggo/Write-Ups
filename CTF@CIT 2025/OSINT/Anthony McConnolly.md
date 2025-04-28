# About

This OSINT (Open Source Intelligence) category was filled with challenges aimed at uncovering more information about Anthony McConnolly. The tasks ranged from finding API keys and domain registrars to identifying specific details from social media posts and public profiles. Below is a detailed explanation of how each challenge was solved.

---

# No Country for Old Keys

## About

**Challenge:** What is Anthony McConnolly's API key?

![](Images/Pasted%20image%2020250428085842.png)
### Solve

The first step in solving this challenge was to find Anthony McConnolly’s GitHub profile. This was achieved by searching for his public GitHub account:

[Anthony McConnolly's GitHub](https://github.com/antmcconn)

On his GitHub, we found an API key hidden in his commit history, which was an important clue. Additionally, his username **antmcconn** was visible, which gave us another lead to work with for further investigation.

Here’s an image from his commit history where the API key is visible:

![](Images/Pasted%20image%2020250425220104.png)


We then used the **instantusername** lookup tool to search for his username **antmcconn**, which led to more information about him and helped us progress with additional clues.

[Instantusername Lookup for antmcconn](https://instantusername.com/?q=antmcconn)

---

# The Domain Always Resolves Twice

## About

**Challenge:** What is Anthony McConnolly's favorite domain registrar?

![](Images/Pasted%20image%2020250428085849.png)
### Solve

The key to solving this challenge was Anthony McConnolly’s LinkedIn profile, where he shared a post about his domain **ippsec.rocks**.

[Anthony McConnolly’s LinkedIn](https://www.linkedin.com/in/anthony-mcconnolly-b9110a351/)

Here’s the relevant post from LinkedIn:

![](Images/Pasted%20image%2020250425221328.png)

Using this information, we performed a **WhoIs lookup** on the domain **ippsec.rocks** and discovered that the domain registrar is **GoDaddy**. This was the correct answer for the challenge.

---

# Throwback to the Future

## About

**Challenge:** What date was Anthony McConnolly's "throwback" photo taken? (Not the day it was posted)

![](Images/Pasted%20image%2020250428085907.png)
### Solve

To answer this, we looked at Anthony McConnolly’s **Twitter** profile, where he had posted a "throwback" photo.

[Anthony McConnolly’s Twitter](https://x.com/antmcconn)

In the post, we can see the photo was taken during the 2023 season of the NFL, with a game between the **Buffalo Bills** and the **New England Patriots**. After identifying the game, we researched the exact date of the match, which was **October 22nd, 2023**.

Here’s an image from his Twitter post showing the throwback photo:

![](Images/Pasted%20image%2020250425220904.png)

---

# The Shawshank Infection

## About

**Challenge:** Anthony McConnolly is researching a specific malware. What command and control endpoint is used by that malware to upload device information?

![](Images/Pasted%20image%2020250428085928.png)
### Solve

We found Anthony McConnolly’s **LinkedIn** profile, where he shared an article about crypto-stealing apps in the Apple App Store. From there, we were able to gather important clues for solving this challenge.

[Anthony McConnolly’s LinkedIn](https://www.linkedin.com/in/anthony-mcconnolly-b9110a351/)

In the article, the malware SDK is described as uploading information about infected devices to a command and control server. Specifically, it uploads data along the path `/api/e/d/u`. The command and control endpoint is identified as **/api/e/d/u**.

Here’s the relevant information from the article that we found on **BleepingComputer**:

[Article on BleepingComputer](https://www.bleepingcomputer.com/news/mobile/crypto-stealing-apps-found-in-apple-app-store-for-the-first-time/)

```
"Then, the SDK uploads information about the device to the command server along the path / api / e / d / u, and in response, receives an object that regulates the subsequent operation of the malware."
```

This led us to the final flag:

```
CIT{/api/e/d/u}
```