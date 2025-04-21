I use a plastic card and pin to log in for work. I seem to have forgotten it and the process to change it is a real hassle. Luckily I capture the USB interfaces on my computer for this very reason! Can you recover my pin from this capture? Wrap the pin in DawgCTF{}

Given a pcap capture of the USB communications.

Since we are working with a smartcard probably using the **ISO/IEC 7816** standard, and we have the captured USB traffic (presumably via a USB PCSC reader or a virtual smartcard reader), you're likely looking at **APDU** (Application Protocol Data Unit) communication over USB.

ISO 7816 smartcard communication follows a **command/response** model:
- Commands are **C-APDUs** (Command APDUs) sent to the smartcard.
- Responses are **R-APDUs** returned by the card.

A **VERIFY** command (`0x20`) is typically used to check a PIN.
```
00 20 00 80 <Lc> <PIN bytes>
```
# Solve
```
frame contains 00:20:00:80
```
![](../Images/Pasted%20image%2020250419221726.png)

![](../Images/Pasted%20image%2020250419221745.png)

At the end we see the PIN value 7901.

```
DawgCTF{7901}
```
