# About
Our Intel team has flagged a suspicious employee at one of our mining sites. We intercepted SMS traffic between his laptop and a connected modem. Your mission: analyze the capture file. Determine whether this is just routine activity from an ordinary worker—or if we’re dealing with a spy hiding in plain sight

# Solve
We are given a pcap file of USB communications

![](Images/Pasted%20image%2020250523144645.png)

Common At command for SMS
```
AT+CMGF=1	Set SMS mode to text (easier to read)
AT+CMGF=0	Set SMS mode to PDU (encoded)
AT+CMGS	Send SMS message
AT+CMGL	List messages
AT+CMGR	Read message
```

We see on line 17 the request to send a message followed by the > prompting the user for input.

When we take the byte stream we get

```
3131303030423931333133333333333333334637303030383030414330303530303036313030373930303643303036463030363130303634303032303030363930303645303037343030363530303732303036333030363530303730303037343030363530303634303032433030323030303438303035343030343230303742303036343030333330303333303037303030354630303331303036453030354630303335303036443030333530303744303032303030353430303638303036353030373930303237303037323030363530303230303037353030373330303639303036453030363730303230303036393030373430303230303037343030364630303230303036323030373930303730303036313030373330303733303032303030363430303635303037343030363530303633303037343030363930303646303036453030324530303230303034443030364630303732303036353030323030303733303036463030364630303645303032451a
```

This is a raw PDU in Unicode that represents a full SMS message. Lets try and decode it, it was double hex encoded. Putting it in cyber chef twice gets us readable information

![](Images/Pasted%20image%2020250523145531.png)


```
HTB{d33p_1n_5m5}
```