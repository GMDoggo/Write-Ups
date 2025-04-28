# About
You were supposed to be a hero, Brian!
![](../Images/Pasted%20image%2020250428105221.png)
# Solve
```
zsteg ChickenJockey.png 
b1,r,lsb,xy         .. text: "v6RNA]bgD"
b1,rgb,lsb,xy       .. text: "VEhJU19pc19hX2NyYWZ0aW5nX3RhYmxl"
b2,b,msb,xy         .. text: "UUUUTU@UU"
b2,rgb,msb,xy       .. file: OpenPGP Public Key
b2,rgba,lsb,xy      .. text: ["#" repeated 8 times]
b2,abgr,msb,xy      .. text: ["S" repeated 8 times]
b3,g,msb,xy         .. file: OpenPGP Secret Key
b4,r,lsb,xy         .. text: "DB\"\"\"\"\"\""
b4,r,msb,xy         .. text: "\"BDDDDDD"
b4,g,lsb,xy         .. text: "\"\"DD\"\"\"\""
b4,g,msb,xy         .. text: "DD\"\"DDDD"
b4,b,lsb,xy         .. text: "DDDDDDDDDF"
b4,b,msb,xy         .. text: "@\"\"\"\"\"\"\"\"\"bU53s3333fffff\"\"\"DDD$\"BDD\"\"\"\"\"\"\"\"DDDD"

```

Flag is B64 encoded in the LSB in RGB Channels.

```
VEhJU19pc19hX2NyYWZ0aW5nX3RhYmxl

THIS_is_a_crafting_table
```
