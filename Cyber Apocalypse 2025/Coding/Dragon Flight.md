```python
def max_subarray_sum(arr):
    """
    Calculates the maximum contiguous subarray sum using Kadane's Algorithm.
    """
    if not arr:  # If the array is empty, return 0.
        return 0

    max_so_far = 0  # Initialize the maximum sum found so far.
    max_ending_here = 0  # Initialize the maximum sum ending at the current position.
    max_element = float('-inf')  # Initialize the maximum single element.

    for num in arr:
        max_ending_here = max(0, max_ending_here + num)  # Update the maximum sum ending here.
        max_so_far = max(max_so_far, max_ending_here)  # Update the overall maximum sum.
        max_element = max(max_element, num)  # Update the maximum single element.

    if max_so_far == 0:  # If no positive sum was found, return the maximum element.
        return max_element
    return max_so_far  # Return the maximum sum found.

def solve_dragon_flight():
    """
    Solves the Dragon Flight problem.
    """
    n, q = map(int, input().split())  # Read N (segments) and Q (operations).
    winds = list(map(int, input().split()))  # Read the initial wind effects.

    for _ in range(q):  # Iterate through each operation.
        operation = input().split()  # Read the operation and split it.
        if operation[0] == 'U':  # If the operation is an update.
            index = int(operation[1]) - 1  # Get the index (0-based).
            value = int(operation[2])  # Get the new wind effect value.
            winds[index] = value  # Update the wind effect at the given index.
        elif operation[0] == 'Q':  # If the operation is a query.
            left = int(operation[1]) - 1  # Get the left index (0-based).
            right = int(operation[2]) - 1 # Get the right index (0-based).
            subarray = winds[left:right + 1]  # Extract the subarray.
            print(max_subarray_sum(subarray))  # Calculate and print the max subarray sum.

solve_dragon_flight()  # Call the solve function.
```