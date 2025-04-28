# About
I made this program, you just have to ask really nicely for the flag!
![](../Images/Pasted%20image%2020250428104444.png)
## Solve
The main portion of the binary we care about is the following
```
{
  bool bVar1;
  ostream *poVar2;
  long in_FS_OFFSET;
  string local_68 [32];
  string local_48 [40];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  poVar2 = std::operator<<((ostream *)std::cout,"How badly do you want the flag?");
  std::ostream::operator<<(poVar2,std::endl<>);
  std::string::string(local_68);
                    /* try { // try from 00407df6 to 00407e93 has its CatchHandler @ 00407f0e */
  std::getline<>((istream *)&std::cin,local_68);
  poVar2 = std::operator<<((ostream *)std::cout,"Ask nicely...");
  std::ostream::operator<<(poVar2,std::endl<>);
  std::getline<>((istream *)&std::cin,local_68);
  bVar1 = std::operator==(local_68,
                          "pretty pretty pretty pretty pretty please with sprinkles and a cherry on top"
                         );
  if (bVar1) {
    poVar2 = std::operator<<((ostream *)std::cout,"Good job, I\'m so proud of you!");
    std::ostream::operator<<(poVar2,std::endl<>);
    std::string::string(local_48,local_68);
                    /* try { // try from 00407e9b to 00407e9f has its CatchHandler @ 00407efd */
    give_flag(local_48);
    std::string::~string(local_48);
  }
  else {
                    /* try { // try from 00407ec2 to 00407ed8 has its CatchHandler @ 00407f0e */
    poVar2 = std::operator<<((ostream *)std::cout,"that\'s not quite what I\'m lookng for.");
    std::ostream::operator<<(poVar2,std::endl<>);
  }
  std::string::~string(local_68);
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
```

Where we can see that it prompts us for input so on so forth. But we can see there is a comparison for the second input against a hardcoded string and if the value returns true it prints the flag.

```
pretty pretty pretty pretty pretty please with sprinkles and a cherry on top

CIT{2G20kX09yF3F}
```