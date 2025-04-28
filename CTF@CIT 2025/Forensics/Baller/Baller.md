# About
![](../Images/Pasted%20image%2020250428082626.png)
# Solve
Using Binwalk we see that there is a gif file stored in this archive.
![](../Images/Pasted%20image%2020250428082720.png)Lets carve that out. This will extract everything from byte 16631 to the end

`dd if=baller.zip of=carved.gif bs=1 skip=16631`

![](../Images/Pasted%20image%2020250425232504.png)

Appears we have a string in the bottom right "im_balling_fr"

```
CIT{im_balling_fr}
```