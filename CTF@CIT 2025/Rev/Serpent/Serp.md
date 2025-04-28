# About

I made this obfuscator a few years ago for my intro to python class. A few features broke but it still works enough, hope u guys like it :)

![](../Images/Pasted%20image%2020250428093348.png)
# Solve

This script is heavily obfuscated through several techniques:
- Confusing variable names (IllIIl style naming)
- Complex mathematical expressions that evaluate to simple integers
- Multiple layers of encoding and transformation
- Weird string concatenation patterns
## Main Functions Overview
1. `IlIIIlIIIIIlI()` - Creates a hash from a string
2. `IIllIIIllIIlIIIIlIl()` - Recursive function for value transformation
3. `Transformer` class - Performs various operations on data
4. `IIllIllIIIIIIlIl()` - Processes a list of values
5. `lIllIlIIlIlllllIllll()` - Seems to be generating a key
6. `IIIIlIIlIll()` - Decodes a string
7. `IllIlllIIllIlIIIIIlI()` - Main function called at the end

Lets do some cleanup using VIM.
```
:%s/IllIlllIIllIlIIIIIlI/stage1/g
:%s/lIllIlIIlIlllllIllll/generate_encoded_string/g
:%s/IIIIlIIlIll/decode_string/g
```

The code does not have any functional prints for the stage1 function

![](../Images/Pasted%20image%2020250428095136.png)

We can easily append that on the end and we are able to print the flag.

![](../Images/Pasted%20image%2020250428095205.png)
```
CIT{7777aKMpU9X3TqnD}
```