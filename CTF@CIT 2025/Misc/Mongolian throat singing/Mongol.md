# About
![](../Images/Pasted%20image%2020250428084733.png)



# Solve
![](../Images/Pasted%20image%2020250428084619.png)
In this challenge you are prompt with various questions and if you fail a question it terminates the session. I was able to build a question bank by quickly scripting it out in python. After building the question bank the rest was just making sure the answers were correct.



```
from pwn import *


REMOTE_HOST = "23.179.17.40"
REMOTE_PORT = 5393


response_map = {
    "Which famous Mongol khanate ruled over much of Russia, Ukraine, and parts of Central Asia during the 13th and 14th centuries?": "Golden Horde",
    "In what year did Mongolia join the World Trade Organization (WTO)?": "1997",
    "Which group do the majority of Kazakhs from South Kazakhstan belong to?": "Senior Zhuz",
    "Who was the founder of the Mongol Empire?": "Genghis Khan",
    "Which population migration is responsible for bringing the majority of Y-chromosomal lineages in South Kazakhstan?": "Niru'un Mongols",
    "Which Mongol leader was known for his conquest of the Song Dynasty in China?": "Kublai Khan",
    "What is the second-largest export product of Mongolia after coal?": "copper",
    "Which Mongol leader attempted to invade Japan twice, in 1274 and 1281, but was thwarted by powerful typhoons, known as the 'kamikaze' winds?": "Kublai Khan",
    "What is the name of the wild camel species found in Mongolia?": "Bactrian Camels",
    "Which animal is the primary source of cashmere in Mongolia?": "cashmere goats",
    "What is Mongolia's primary export commodity?": "coal",
    "What important Mongol battle occurred in 1241 that delayed the Mongol invasion of Europe?": "The Battle of Mohi",
    "Which famous Mongol general and grandson of Genghis Khan is known for leading the conquest of the Ilkhanate in Persia?": "Hulegu Khan",
    "What is the name of the book that Marco Polo wrote about his travels to the Mongol Empire?": "The Travels of Marco Polo",
    "Which famous traveler from Venice visited the Mongol Empire during the reign of Kublai Khan?": "Marco Polo",
    "What is the official currency of Mongolia?": "tugrik",
    "In which year did the Mongol Empire officially split into four khanates?": "1260",
    "Which species of wild goat in Mongolia is famous for its badass horns?": "Markhor",
    "What animal is used extensively by Mongolian herders for milk, wool, and meat?": "yak",
}


context.log_level = "info"


p = None
try:
    log.info(f"Connecting to {REMOTE_HOST}:{REMOTE_PORT}")
    p = remote(REMOTE_HOST, REMOTE_PORT)

    while True:
        try:
            received_bytes = p.recvline(timeout=2)

            if not received_bytes:
                log.warning("Received empty bytes. Connection likely closed by server.")
                break

            received_string = received_bytes.decode(errors="ignore").strip()

            if not received_string:
                continue

            log.info(f"Received: '{received_string}'")

            if received_string in response_map:
                response_to_send = response_map[received_string]
                log.success(f"Recognized key. Sending: '{response_to_send}'")
                p.sendline(response_to_send.encode())
            else:
                pass

        except EOFError:
            log.info("Connection closed by remote host (EOF).")
            break
        except Exception as e:
            log.error(f"An error occurred during interaction: {e}")
            import traceback

            traceback.print_exc()
            break

except ConnectionRefusedError:
    log.error(
        f"Connection refused by {REMOTE_HOST}:{REMOTE_PORT}. Is the service running?"
    )
except Exception as e:
    log.error(f"Failed to connect or other setup error: {e}")
    import traceback

    traceback.print_exc()

finally:
    if p and p.connected():
        log.info("Closing connection.")
        p.close()
    elif p and not p.connected():
        log.info("Connection already closed.")
    else:
        log.info("Connection was not established.")
```