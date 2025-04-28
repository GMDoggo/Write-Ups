# About

![](../Images/Pasted%20image%2020250428104048.png)

# Solve
I know there was probably a proper way to solve this challenge, but in all honesty I most certainly did this the unintended way. Since we had the local binary and we know it would output CIT{} I basically just made a python script to just bruteforce this file.

Luckily enough I just wanted to start with numerical values 1-10000
```
#!/usr/bin/env python3
import subprocess
import os

# Path to the binary
BINARY_PATH = os.path.expanduser("~/Downloads/securekeygen")

def try_key(key):
    """
    Attempt to run the binary with the given key and return the output.
    """
    try:
        # Run the binary and provide the key as input
        process = subprocess.run(
            [BINARY_PATH],
            input=str(key),
            text=True,
            capture_output=True,
            timeout=1  # Timeout to prevent hanging
        )
        # Return the output (stdout and stderr)
        return process.stdout + process.stderr
    except subprocess.TimeoutExpired:
        return "Timeout"
    except Exception as e:
        return f"Error: {str(e)}"

def main():
    # Test numerical values from 1 to 10000
    for key in range(1, 10001):
        print(f"Trying key: {key}", end="\r")  # Show progress
        output = try_key(key)
        
        # Check if "CIT" is in the output
        if "cit" in output.lower():
            print(f"\nFound 'CIT'! Key: {key}")
            print(f"Output: {output}")
            break
    else:
        print("\nNo key produced 'CIT' in range 1-10000.")

if __name__ == "__main__":
    # Verify the binary exists
    if not os.path.isfile(BINARY_PATH):
        print(f"Error: Binary not found at {BINARY_PATH}")
    else:
        main()
```

Coincidentally we were able to get the flag.

![](../Images/Pasted%20image%2020250428104328.png)

```
CIT{w4tUcLj95fpq}
```