#### Description
Can you get the flag?
Go to this [website](http://saturn.picoctf.net:52043/) and see what you can discover.
## Solve
On our main page there is not much besides a login page
![](Images/Pasted%20image%2020250330214840.png)
When we take a look at the login.php we can further see how this process works.
![](Images/Pasted%20image%2020250330214918.png)
So when looking at the login.php file we see that on a successful login it sets our hash to the Admin Hash.
The client side appears to be handled by JavaScript.
Looking further, in the Secure.js folder is the admin credentials in plaintext
![](Images/Pasted%20image%2020250330220008.png)
