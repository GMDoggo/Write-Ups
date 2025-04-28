# About
For this challenge we were provided a cipher text which was a Substitution cipher. Using python I was able to decode the entire block using my knowledge of the flag format being `CIT{`

![](Images/Pasted%20image%2020250428081914.png)
# Solve

```
# Python script to decode a monoalphabetic substitution cipher from nerd.txt

# Define the substitution dictionary (ciphertext -> plaintext)

substitution = {

    'A': 'W', 'B': 'X', 'C': 'V', 'D': 'M', 'E': 'C', 'F': 'N', 'G': 'O',

    'H': 'P', 'I': 'H', 'J': 'Q', 'K': 'R', 'L': 'S', 'M': 'Z', 'N': 'Y',

    'O': 'I', 'P': 'J', 'Q': 'A', 'R': 'D', 'S': 'L', 'T': 'E', 'U': 'G',

    'V': 'W', 'W': 'B', 'X': 'U', 'Y': 'F', 'Z': 'T',

    'a': 'w', 'b': 'x', 'c': 'v', 'd': 'm', 'e': 'c', 'f': 'n', 'g': 'o',

    'h': 'p', 'i': 'h', 'j': 'q', 'k': 'r', 'l': 's', 'm': 'z', 'n': 'y',

    'o': 'i', 'p': 'j', 'q': 'a', 'r': 'd', 's': 'l', 't': 'e', 'u': 'g',

    'v': 'w', 'w': 'b', 'x': 'u', 'y': 'f', 'z': 't'

}

  

def decode_ciphertext(ciphertext):

    """Decode the ciphertext using the substitution table, preserving non-letters."""

    decoded = ""

    for char in ciphertext:

        # Apply substitution if the character is in the table, otherwise keep it

        decoded += substitution.get(char, char)

    return decoded

  

def main():

    try:

        # Read the ciphertext from nerd.txt

        with open('nerd.txt', 'r', encoding='utf-8') as file:

            ciphertext = file.read()

        # Decode the ciphertext

        plaintext = decode_ciphertext(ciphertext)

        # Print the decoded text

        print("Decoded Plaintext:\n")

        print(plaintext)

        # Extract and decode the flag (assuming it's at the end in the format EOZ{...})

        import re

        flag_match = re.search(r'EOZ\{[A-Za-z]+\}', ciphertext)

        if flag_match:

            flag = flag_match.group(0)

            decoded_flag = decode_ciphertext(flag)

            print("\nDecoded Flag:")

            print(decoded_flag)

    except FileNotFoundError:

        print("Error: nerd.txt not found. Please ensure the file exists in the same directory.")

    except Exception as e:

        print(f"Error: {str(e)}")

  

if __name__ == "__main__":

    main()
```


```
CIT{XSEFLHTRCHQABDGSDW}
```