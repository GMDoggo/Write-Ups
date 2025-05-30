# About
Volnayan APTs are exfiltrating data through TOR nodes, embedding attack signals in plain sight. Your job is to scan each outbound stream and identify known malicious keywords linked to Operation Blackout. Each keyword has a threat level — the more hits you find, the higher the danger. Analyze the stream, tally the signals, and calculate the overall threat score.

## Problem Requirements
- **Input:** A data stream (string) containing only lowercase letters and digits, with a length between 30 and 10^6.
- **Task:** Calculate a threat score based on the frequency of specific keywords in the stream, where each keyword has an associated weight.
- **Formula:** Threat score = Σ (occurrences of keyword × keyword weight).
- **Keywords and Weights:** A predefined list of 18 keywords (e.g., "scan" = 1, "payload" = 15, "botnet" = 18, etc.).
- **Output:** A single integer representing the total threat score.
- **Example:** For input payloadrandompayloadhtbzerodayrandombytesmalware, the expected output is 60 (2 × 15 for "payload" + 1 × 17 for "zeroday" + 1 × 13 for "malware").

```
# Define the keywords and their weights
keywords = {
    "scan": 1,
    "response": 2,
    "control": 3,
    "callback": 4,
    "implant": 5,
    "zombie": 6,
    "trigger": 7,
    "infected": 8,
    "compromise": 9,
    "inject": 10,
    "execute": 11,
    "deploy": 12,
    "malware": 13,
    "exploit": 14,
    "payload": 15,
    "backdoor": 16,
    "zeroday": 17,
    "botnet": 18
}

# Take in the data stream
n = input()

# Calculate the threat score
threat_score = 0
for keyword, weight in keywords.items():
    # Count non-overlapping occurrences of the keyword in the stream
    count = n.count(keyword)
    # Add to threat score: occurrences * weight
    threat_score += count * weight

# Print the threat score
print(threat_score)
```

## Explanation
- A variable threat_score is initialized to 0 to store the cumulative threat score.
- The code iterates over the keywords dictionary using .items(), which provides both the keyword (e.g., "payload") and its weight (e.g., 15) in each iteration.
- For each keyword:
    - The string method n.count(keyword) counts the number of non-overlapping occurrences of the keyword in the input stream n. For example, in the string "payloadrandompayload", "payload" appears twice.
    - The count is multiplied by the keyword’s weight, and the result is added to threat_score.
- This implements the formula: threat score = Σ (occurrences of keyword × keyword weight), summing the contributions of all keywords.