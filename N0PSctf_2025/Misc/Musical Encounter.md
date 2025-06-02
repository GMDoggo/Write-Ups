# Problem Description
n00psy takes you to a nature walk in N0PStopia. On your way, you meet a group of dancing skeletons! What an amazing adventure to see and hear. I wonder what they talk about...

# Solve
My first gut instinct was this was going to be a steg problem where there is a hidden track that contains a message. lets go ahead and try that first.

We can strip out the audio track with the following command

```
ffmpeg -i dancing_skeletons.mp4 -vn -acodec copy audio.aac
```

Opening in audacity spectrogram we find the flag.

![](Images/Pasted%20image%2020250531103413.png)

```
N0PS{c4rT5s0N8z}
```

