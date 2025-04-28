# About
![](../Images/Pasted%20image%2020250428093412.png)


main
```
undefined8 main(void)
{
  long in_FS_OFFSET;
  string local_68 [32];
  string local_48 [40];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  std::string::string(local_68);
                    /* try { // try from 0040802d to 0040805a has its CatchHandler @ 004080a8 */
  std::operator<<((ostream *)std::cout,"Enter key: ");
  std::getline<>((istream *)&std::cin,local_68);
  std::string::string(local_48,local_68);
                    /* try { // try from 00408062 to 00408066 has its CatchHandler @ 00408097 */
  check_key(local_48);
  std::string::~string(local_48);
  std::string::~string(local_68);
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```


- Prompts the user for input.
- Reads a string (the "key") from standard input.
- Passes the string to a function called `check_key`.
- Does some stack protection validation (stack canary).
- Cleans up memory.

``` check_key
/* check_key(std::string) */

undefined8 check_key(string *param_1)

{
  bool bVar1;
  int iVar2;
  long lVar3;
  char *pcVar4;
  long in_FS_OFFSET;
  ulong local_50;
  string local_48 [40];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  lVar3 = std::string::length(param_1);
  if (lVar3 == 0x10) {
    std::string::substr((ulong)local_48,(ulong)param_1);
                    /* try { // try from 00407ee9 to 00407eed has its CatchHandler @ 00407f8e */
    bVar1 = std::operator!=(local_48,"KEY_");
    std::string::~string(local_48);
    if (!bVar1) {
      for (local_50 = 4; local_50 < 0x10; local_50 = local_50 + 1) {
        pcVar4 = (char *)std::string::operator[](param_1,local_50);
        iVar2 = isalnum((int)*pcVar4);
        if (iVar2 == 0) goto LAB_00407f7d;
      }
      std::string::string(local_48,param_1);
                    /* try { // try from 00407f67 to 00407f6b has its CatchHandler @ 00407fbc */
      validate(local_48);
      std::string::~string(local_48);
    }
  }
LAB_00407f7d:
  if (local_20 == *(long *)(in_FS_OFFSET + 0x28)) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}

```

Key Validation Criteria
Based on the code, the input string must satisfy the following:
1. **Length**: Exactly 16 characters.
2. **Prefix**: Likely starts with "KEY_" (though the substr call is ambiguous).
3. **Characters 5–16**: All characters from index 4 to 15 must be alphanumeric (letters A–Z, a–z, or digits 0–9).
4. **Validation**: Must pass the validate function (details unknown).

An example input would look like this
```
KEY_ABC123XYZ789
```

The problem was in testing, no matter what key I input as long as it matched the criteria it didn't need to pass the validate function

![](../Images/Pasted%20image%2020250428102509.png)

```
CIT{41jN8BKzz388}
```