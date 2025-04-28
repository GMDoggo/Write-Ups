# About
Sorry everyone, we unfortunately lost the flag for this challenge.
![](../Images/Pasted%20image%2020250428082127.png)
## Solve
Doing a quick `file lost.png` shows that this is not a valid png file.
```
┌──(kali㉿kali)-[~/Downloads]
└─$ file lost.png       
lost.png: data
                                                     
┌──(kali㉿kali)-[~/Downloads]
└─$ xxd lost.png | head
00000000: 00c2 ba60 0010 4a46 4946 0001 0101 0048  ...`..JFIF.....H
00000010: 0048 0000 ffdb 0043 000a 0707 0807 060a  .H.....C........
00000020: 0808 080b 0a0a 0b0e 1810 0e0d 0d0e 1d15  ................
00000030: 1611 1823 1f25 2422 1f22 2126 2b37 2f26  ...£.%$"."!&+7/&
00000040: 2934 2921 2230 4131 3439 3b3e 3e3e 252e  )4)!"0A149;>>>%.
00000050: 4449 433c 4837 3d3e 3bff db00 4301 0a0b  DIC<H7=>;...C...
00000060: 0b0e 0d0e 1c10 101c 3b28 2228 3b3b 3b3b  ........;("(;;;;
00000070: 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b  ;;;;;;;;;;;;;;;;
00000080: 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b  ;;;;;;;;;;;;;;;;
00000090: 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b 3b3b ffc0  ;;;;;;;;;;;;;;..
```
This challenge has to do with the bad header as its a png file but the headers shows its a .jpg.

I tried just renaming it to .jpg and that also didn't work so I have to restore the header manually.

I grabbed a jpg online for easy replacement.
```
head -c 20 scan.jpg > jpeg_header.bin
cat jpeg_header.bin <(tail -c +21 lost.jpg) > fixed.jpg
file fixed.jpg     
fixed.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 4077x4000, components 3
```
![](../Images/Pasted%20image%2020250426002522.png)






