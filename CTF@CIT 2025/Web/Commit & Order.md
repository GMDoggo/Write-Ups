# About
"In software development, the repository is represented by two separate yet equally important branches..."
![](Images/Pasted%20image%2020250428110905.png)
# Solve
Based on the challenge name and description I knew this was going to be something pertaining to git.

Sure enough we can dump the .git and use extractor to grab the source files.

![](Images/Pasted%20image%2020250426113217.png)

![](Images/Pasted%20image%2020250426113258.png)

After searching for a while I found the admin password within the index.php but it doesn't lead anywhere. 

After beating my head I realized in the header was a B64 encoded string in one of the commits to admin.php

![](Images/Pasted%20image%2020250426113404.png)

Decoding that gets us the flag.

![](Images/Pasted%20image%2020250426113420.png)

