Our interns put together a little test program for us. It seems they all might have patched together their separate projects. Could you test it out for me?

nc connect.umbccd.net 20011


# Solve
```
= geteuid(), _Var2 != 0)) {
    bVar1 = true;
  }
  else {
    bVar1 = false;
  }
  if (bVar1) {
    poVar4 = std::operator<<((ostream *)std::cout,"Error: Option 2 requires root privileges HAHA");
    std::ostream::operator<<(poVar4,std::endl<>);
```
The check for printFlag trigger only matters if option 2 is selected first.
```
└─$ echo "1 2" | nc connect.umbccd.net 20011
Welcome to our intern's test project!

The following are your options:
   1. Say hi
   2. Print the flag
   3. Create an account
Enter option (1-3). Press Enter to submit:
Hi!
Here is your flag: DawgCTF{B@d_P3rm1ssi0ns}
```