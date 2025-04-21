# About
In this challenge, you’ll need to uncover a hidden secret, but first, you must find the key. The key is hidden in plain sight. A user named Wikikenobi has left you a breadcrumb trail, and it’s your job to follow it. Once you’re there, look closely at everything.

aiye_hoav_aqd_advi

(Enclose the decoded message in DawgCTF{})

# Solve
Finding the user on Wikipedia
![](Images/Pasted%20image%2020250419124030.png)
In the prompt we have to look in "plain sight" and decode message aiye_hoav_aqd_advi

If we take the first letter of sentence, we get P A D A W A N. If we use this in decoding, we can find it's a Vigenere Cipher with the key being PADAWAN.

This decodes to 
```
live_long_and_edit
```



