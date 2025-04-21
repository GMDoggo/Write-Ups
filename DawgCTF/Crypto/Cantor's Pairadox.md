# About
Now that I have encrypted my flag with a new math function I was just researching I can know share it with my friend Cantor and no one will know how to read it except us!

## Source
```python
from sage.all import sqrt, floor
from secret import flag

def getTriNumber(n):
    return n * (n + 1) // 2  # Ensure integer division

def pair(n1, n2):
    S = n1 + n2
    return getTriNumber(S) + n2

def pair_array(arr):
    result = []
    for i in range(0, len(arr), 2):
        result.append(pair(arr[i], arr[i + 1]))    
    return result

def pad_to_power_of_two(arr):
    result = arr
    n = len(result)
    while (n & (n - 1)) != 0:
        result.append(0)
        n += 1
    return result
    
flag = [ord(f) for f in flag]  
flag = pad_to_power_of_two(flag)

temp = flag
for i in range(6):
    temp = pair_array(temp)

print("Encoded:", temp)
```
```
flag = 4036872197130975885183239290191447112180924008343518098638033545535893348884348262766810360707383741794721392226291497314826201270847784737584016
```

## Overview of the Process

The encoding process is performed in a series of transformations using triangular numbers and a custom pairing function. Below is a detailed breakdown of how the code works and how the correct encoded flag is generated.
## Step-by-Step Breakdown

### 1. **`getTriNumber(n)` function**

The `getTriNumber()` function computes the **triangular number** of an integer `n`. The triangular number is the sum of the first `n` integers.
This function is central to the encoding process, as it helps create the encoding by calculating triangular numbers.
### 2. **`pair(n1, n2)` function**

The `pair()` function takes two integers, `n1` and `n2`, and performs the following steps:
- It computes their sum `S = n1 + n2`.
- It then computes the triangular number of `S` using the `getTriNumber()` function.
- Finally, it adds `n2` to the result and returns it

This operation encodes each pair of numbers from the input array.

### 3. **`pair_array(arr)` function**

The `pair_array()` function processes an array of integers in pairs. It splits the array into two-element chunks, applies the `pair()` function to each pair, and stores the results in a new array. For example, if the input is an array of length 4, it will produce an array of length 2 after applying `pair()` to each pair.
### 4. **`pad_to_power_of_two(arr)` function**

The `pad_to_power_of_two()` function ensures that the input array has a length that is a power of two. If the length is not already a power of two, the function pads the array with zeros until the length satisfies this condition. This step is important for the encoding process, as some algorithms work most efficiently when the length of the array is a power of two.
### 5. **Flag Encoding Process**

The encoding process begins with the flag string:
- The flag is converted into a list of its **ASCII values**. Each character of the flag is represented by its corresponding integer value.
- The list is padded to a power of two in length using the `pad_to_power_of_two()` function.
- The padded list is then repeatedly transformed using the `pair_array()` function. The transformation is applied 6 times in total, progressively encoding the original list into a new form.
  
After each iteration, the size of the array reduces by half. The result is a new array that represents the flag in its encoded form.

```
from sage.all import sqrt, floor
from secret import flag

def getTriNumber(n):
    return n * (n + 1) // 2  # Ensure integer division

def pair(n1, n2):
    S = n1 + n2
    return getTriNumber(S) + n2

def pair_array(arr):
    result = []
    for i in range(0, len(arr), 2):
        result.append(pair(arr[i], arr[i + 1]))    
    return result

def pad_to_power_of_two(arr):
    result = arr
    n = len(result)
    while (n & (n - 1)) != 0:
        result.append(0)
        n += 1
    return result
    
flag = [ord(f) for f in flag]  
flag = pad_to_power_of_two(flag)

temp = flag
for i in range(6):
    temp = pair_array(temp)

print("Encoded:", temp)
```

```
Dawg{1_pr3f3r_4ppl3s_t0_pa1rs_4nyw2y5}
```