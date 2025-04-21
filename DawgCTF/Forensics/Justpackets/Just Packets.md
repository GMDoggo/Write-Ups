# About
Here pcap. Find flag.
# Solve
Decrypted FTP stream was just a bunch of garbage fluff text.
```
Once upon a time, in a small, vibrant village nestled between rolling hills and vast forests, a mysterious legend was whispered among the people. The legend spoke of an ancient flag, hidden away for centuries, said to grant immense wisdom and fortune to the one who found it. This flag was not just a piece of fabric; it was a symbol of the village's history and secrets, woven into its very threads.

Year after year, curious adventurers and determined locals set out to find this elusive flag. They scoured ancient texts, deciphered cryptic clues, and embarked on daring expeditions into the unknown parts of the forest. But no matter how hard they searched, the flag remained hidden, as if it was just a figment of their collective imagination.

Among these seekers was a young girl named Elara. Unlike others, Elara was not driven by the promise of wisdom or fortune. She was captivated by the stories and the history that the flag represented. She spent her days poring over old books and listening to the tales of the elders. Her heart was set on finding the flag, not for the glory, but to understand the stories it held within its fibers.

One starlit evening, as Elara sat reading an ancient manuscript, she stumbled upon a line that struck her differently. It read, "The flag you seek is not in this story at all. It'd be a waste of time analysing this text further (seriously)." Puzzled, Elara pondered over these words. She realized that the flag was never meant to be a physical object to be discovered in her story. Instead, maybe it was a metaphor for the village's rich history and the stories that bound its people together.

Elara shared her revelation with the villagers. They gathered around, listening intently as she spoke of the journeys they had undertaken in search of the flag and the bonds they had formed along the way. The stories of their ancestors, their own adventures, and the lessons they learned were the true flag of their village. It was not something to be found but something to be lived and passed down through generations.

From that day on, the villagers no longer sought the flag in the forests or ancient ruins. They found it in their everyday lives, in the stories they shared, and in the legacy they would leave for future generations. The flag was in their heart, in their stories, a text that did not need to be written but to be lived and cherished. And so, the village continued to thrive, rich in tales and wisdom, with the flag forever waving in their collective spirit.
```

The hint stated to look at the URG pointers
I found this website
https://ctf-wiki.mahaloz.re/misc/traffic/data/
Grab URG Pointers
```
tshark -r traffic.pcap -T fields -e  tcp.urgent_pointer | egrep -vi "^0$" | tr '\n' ','
25697,30567,17236,18043,30313,27756,24935,25970,29535,25199,28260,29565,10,   
```

Since URG pointer is a 16 bit value we have to examine them that way. Let me convert each 16-bit value to its corresponding ASCII representation by looking at potential byte patterns.

For 16-bit values, we could look at:
- The lower 8 bits (value % 256)
- The upper 8 bits (value // 256)
![](Images/Pasted%20image%2020250419190833.png)
There is a bit of formatting issues but the main text container in {} of the flag was correct.
```
DawgCTF{villagers_bonds}
```