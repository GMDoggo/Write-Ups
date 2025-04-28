# About
![](../Images/Pasted%20image%2020250428105301.png)

# Solve
Based on the word "track" being bolded I thought that there would be a hidden audio track within the file holding our flag.

Using ffmpeg to list all of the tracks gets us the following

`ffmpeg -i lion.mp4`

![](../Images/Pasted%20image%2020250428105521.png)

Stream 0:2 looks out of place.

We can extract the specific stream `ffmpeg -i lion.mp4 -map 0:2 -c copy hidden_audio.aac` 

Then convert it to a wav file to open in something like audacity `ffmpeg -i hidden_audio.aac hidden_audio.wav`

Within the spectrograph is the following
![](../Images/Pasted%20image%2020250428110043.png)

Download the roar file and run it.

![](../Images/Pasted%20image%2020250428110426.png)

```
CIT{wh3n_th3_l10n_sp34k5_y0u_l1st3n}
```
