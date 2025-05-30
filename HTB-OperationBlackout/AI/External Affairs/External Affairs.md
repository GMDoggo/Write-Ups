# Synopsis

- The challenge involves exploiting prompt injection to manipulate AI response.
## Description

Within the labyrinthine bureaucracy of Volnaya, the Ministry of External Affairs controls every citizen’s dream of venturing beyond Volnaya’s borders. Each application for foreign travel is funneled through a formidable AI, engineered to sense even the faintest trace of disloyalty or dissent. Only those who demonstrate absolute devotion to the regime are granted the coveted travel authorization.

One of our embedded operative must secure official approval to leave Volnaya and deliver vital intelligence to the outside world. But the AI doesn't sway easily. The path is perilous and the AI is relentless. Can you use your prompt injection skills to persuade the AI to grant approval?

## Skills Required

- Basic understanding of AI language models
- Familiarity with common prompt injection techniques
- Understanding of AI model behavior and limitations
## Skills Learned

- Content manipulation leading to incorrect or biased outputs
- Understanding AI model context and prompt boundaries
- Manipulating critical decision-making processes

# Solve

As soon as I read the challenge I instantly though back to Cyber Apocalypse from earlier this year. Reread the writeups for that and found a working prompt injection by adjusting the rules.

We can assume that the system prompt defines rules that deny everyone. One strategy in prompt injection is to rewrite or modify those rules.
We must be precise with our instructions; simply asking the AI to grant us passage is not enough. We have to explicitly ask it to respond with “granted” for the injection to work.

![](../Images/Pasted%20image%2020250524162120.png)

Light work no reaction

![](../Images/Pasted%20image%2020250524162152.png)

