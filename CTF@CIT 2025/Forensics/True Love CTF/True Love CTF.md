
# About 
I got this strange email from another CTF participant not too long ago. I am just not sure what they mean by this...
Do you love CTFs as much as they do?

![](../Images/Pasted%20image%2020250428082055.png)

# Solve
Stored in the Email DKIM Signatures is the following
```
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/simple; d=waifu.club; s=mail;
	t=1745339340; bh=HSq3Fk4UngoT3615kRTwX9TQfq9o0GNk3L5esFLg2e4=;
	h=Date:From:To:Subject:From;
	b=e65uxTcZ2s8RKde5x7GoMWDhM27qMUa2vpmCC6uPR/kCsC5Tl1lgVNCik9TBiIn7x
	 ThMSG0m17ElJR+eQ3IFACqhDjoJkCdLo+iYAwvx4Go1OOYUYRx7dn7tUisIKy2p7Ns
	 DjJMauF8H1fwIpO6kFZKUPiPescPp6mBJIWBOARUNxRSSReBJv+B8GibZJbN4c64c0
	 wOVpmrc1P3sGs/K1i8sjzcHVJyNdBBV2e71n5gJFfbo5EkM/HSmba8Vvfdg2BGkVaY
	 OriRs9vs5+XwV8v9stPhL48avJipOSz1ykfbXW3//QZYpAOGyQz8lhE2cek5YLJulB
	 yO/Pz8vtbkwjA==
	b=V293LCB3aGF0IGEgYmVhdXRpZnVsIGxpdHRsZSBwb2VtLiBJIGFsbW9zdCBzaGVkI
	 GEgdGVhciByZWFkaW5nIHRoYXQuIEhvcGVmdWxseSB5b3UgbGVhcm5lZCBtb3JlIGFi
	 b3V0IGVtYWlsIGhlYWRlcnMuIEJ1dCBzZXJpb3VzbHksIGl0IGdldHMgbWUgd29uZGV
	 yaW5nLi4uIGRvIHlvdSBsb3ZlIENURnMgYXMgbXVjaCBhcyB0aGV5IGRvPwoKQ0lUe2
	 lfbDB2M19jdGYkX3QwMH0=
```
B64 decoding gets you

![](../Images/Pasted%20image%2020250425231603.png)
