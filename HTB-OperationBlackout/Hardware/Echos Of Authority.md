Our Intel-Team has intercepted a VoIP call made by a high-ranking government official. During the call, the official accessed a secure government IVR (Interactive Voice Response) system, which requires a password to proceed. Your mission is to analyze the captured call and determine the password he entered. Flag format: HTB{ACCESS_KEY}


# Solve

Filter for VOIP `sip || rtp`

using wireshark we can listen or playback the audio.

Use the menu:
- **Telephony → VoIP Calls**
- Identify the active call, select it, then click **"Player"**.
- Phone dial sounds can be decoded based on their frequency

![](Images/Pasted%20image%2020250523212835.png)

Lets export this and see if we can get the access key from the dial tones an example can be found [here](https://www.linkedin.com/pulse/forensic-methods-determine-phone-numbers-called-from-keypad-todd/)

![](Images/Pasted%20image%2020250419123233.png)

Export to audacity Analyze > Plot Spectrum

```
HTB{13513377#}
```


In the following example is the first digit

![](Images/Pasted%20image%2020250523213258.png)

On the first peak we get 698 and on the second we get 1207 so this is probably a 1

![](Images/Pasted%20image%2020250523213326.png)

Then rinse and repeat for all digits. or just use a DTMF analzer found [here](http://dialabc.com/sound/detect/index.html)
