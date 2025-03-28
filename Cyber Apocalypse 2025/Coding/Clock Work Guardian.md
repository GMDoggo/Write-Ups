```python
import ast
from collections import deque

def shortest_safe_path(grid):
    """
    Finds the shortest safe path from (0, 0) to 'E' in a grid, avoiding obstacles (1).

    Args:
        grid (list of lists): The grid representing the spire.

    Returns:
        int: The length of the shortest safe path, or -1 if no path is found.
    """

    rows = len(grid)
    cols = len(grid[0])
    queue = deque([(0, 0, 0)])  # row, col, steps
    visited = set()

    moves = [(1, 0), (0, 1), (-1, 0), (0, -1)]

    while queue:
        row, col, steps = queue.popleft()

        if grid[row][col] == 'E':
            return steps

        if (row, col) in visited:
            continue
        visited.add((row, col))

        for dr, dc in moves:
            new_row, new_col = row + dr, col + dc

            if 0 <= new_row < rows and 0 <= new_col < cols and grid[new_row][new_col] != 1:
                queue.append((new_row, new_col, steps + 1))

    return -1

# Input grid as a string:
grid_str = input()

# Convert the string to a list of lists:
grid = ast.literal_eval(grid_str)

result = shortest_safe_path(grid)
print(result)
```