```python
def decode_message(encoded_message, num_groups, shift_values):
    # Initialize variables
    decoded_message = ""
    alphabet_chars_processed = 0
    current_group = 0
    
    # Process each character in the encoded message
    for char in encoded_message:
        # Check if the character is alphabetical
        if char.isalpha():
            # Determine which group this character belongs to
            group_index = min(current_group, len(shift_values) - 1)
            shift = shift_values[group_index]
            
            # Decode the character by shifting backwards
            if char.islower():
                decoded_char = chr(((ord(char) - ord('a') - shift) % 26) + ord('a'))
            else:
                decoded_char = chr(((ord(char) - ord('A') - shift) % 26) + ord('A'))
            
            decoded_message += decoded_char
            
            # Track progress through alphabet characters
            alphabet_chars_processed += 1
            
            # Check if we've completed a group of 5 alphabet characters
            if alphabet_chars_processed == 5:
                alphabet_chars_processed = 0
                current_group += 1
        else:
            # Non-alphabet characters pass through unchanged
            decoded_message += char
    
    return decoded_message

# Example usage
def main():
    # Read input
    encoded_message = input().strip()
    num_groups = int(input().strip())
    shift_values = eval(input().strip())  # Reads the list of shift values
    
    # Decode and print the result
    result = decode_message(encoded_message, num_groups, shift_values)
    print(result)

if __name__ == "__main__":
    main()
```