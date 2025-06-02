## Challenge Description

Valentine the bee needs to visit all 50 waterlilies around her beehive, starting and ending at the beehive located at coordinates (0,0). Each flower must be visited exactly once, and the total path distance must be less than 1400 units.

This is a classic **Traveling Salesman Problem (TSP)** - find the shortest route that visits all points exactly once and returns to the starting point.

## Solution Approach

Since TSP is NP-hard and we have 50 points, finding the optimal solution would take too long. Instead, I used a combination of heuristics to find a "good enough" solution:

### 1. Multiple Starting Strategies

Instead of always starting from the beehive, I tried:

- Standard nearest neighbor from beehive
- Starting from the flower closest to origin
- Starting from several different flowers randomly

**Why this works**: Sometimes starting the tour from a strategically located flower produces a shorter total path than always beginning at the beehive.

### 2. Nearest Neighbor Heuristic

For each starting approach:

- Begin at chosen starting point
- Always visit the nearest unvisited flower
- Continue until all flowers are visited
- Return to beehive

**Time Complexity**: O(n²) - fast enough for 50 points

### 3. 2-opt Local Optimization

After finding an initial solution, improve it using 2-opt:

- Take any two edges in the current path
- Remove them and reconnect in the alternate way
- If the new path is shorter, keep it
- Repeat until no improvements are found

**Example of 2-opt**:

```
Original: A -> B -> C -> D -> E -> A
Remove edges: B-C and D-E
Reconnect: A -> B -> D -> C -> E -> A
```

This eliminates "crossing" paths that unnecessarily increase distance.

## Code Implementation

```
import math
import random

def distance(p1, p2):
    """Calculate Euclidean distance between two points"""
    return math.sqrt((p1[0] - p2[0])**2 + (p1[1] - p2[1])**2)

def calculate_total_distance(path, points):
    """Calculate total distance for a given path"""
    beehive = (0, 0)
    total = 0
    current_pos = beehive
    
    # Visit each flower in order
    for i in range(1, len(path) - 1):  # Skip first and last 0
        flower_idx = path[i] - 1  # Convert to 0-indexed
        total += distance(current_pos, points[flower_idx])
        current_pos = points[flower_idx]
    
    # Return to beehive
    total += distance(current_pos, beehive)
    return total

def nearest_neighbor_from_start(points, start_flower=None):
    """
    Solve TSP using nearest neighbor heuristic
    Can optionally start from a specific flower instead of beehive
    """
    n = len(points)
    beehive = (0, 0)
    
    if start_flower is None:
        # Traditional approach: start from beehive
        current_pos = beehive
        unvisited = set(range(n))
        path = [0]
        total_dist = 0
        
        while unvisited:
            nearest_idx = None
            nearest_dist = float('inf')
            
            for i in unvisited:
                dist = distance(current_pos, points[i])
                if dist < nearest_dist:
                    nearest_dist = dist
                    nearest_idx = i
            
            unvisited.remove(nearest_idx)
            path.append(nearest_idx + 1)
            total_dist += nearest_dist
            current_pos = points[nearest_idx]
        
        total_dist += distance(current_pos, beehive)
        path.append(0)
        
    else:
        # Start from a specific flower, visit others, return to beehive
        current_pos = points[start_flower]
        unvisited = set(range(n))
        unvisited.remove(start_flower)
        
        path = [0, start_flower + 1]  # beehive -> start_flower
        total_dist = distance(beehive, points[start_flower])
        
        while unvisited:
            nearest_idx = None
            nearest_dist = float('inf')
            
            for i in unvisited:
                dist = distance(current_pos, points[i])
                if dist < nearest_dist:
                    nearest_dist = dist
                    nearest_idx = i
            
            unvisited.remove(nearest_idx)
            path.append(nearest_idx + 1)
            total_dist += nearest_dist
            current_pos = points[nearest_idx]
        
        total_dist += distance(current_pos, beehive)
        path.append(0)
    
    return path, total_dist

def two_opt_improve(path, points, max_iterations=1000):
    """Improve a TSP solution using 2-opt"""
    best_path = path[:]
    best_distance = calculate_total_distance(path, points)
    improved = True
    iterations = 0
    
    while improved and iterations < max_iterations:
        improved = False
        iterations += 1
        
        # Try all possible 2-opt swaps
        for i in range(1, len(path) - 2):
            for j in range(i + 1, len(path) - 1):
                # Create new path by reversing the segment between i and j
                new_path = path[:i] + path[i:j+1][::-1] + path[j+1:]
                new_distance = calculate_total_distance(new_path, points)
                
                if new_distance < best_distance:
                    best_path = new_path[:]
                    best_distance = new_distance
                    improved = True
        
        path = best_path[:]
    
    return best_path, best_distance

# Parse the coordinates
coordinates = [
    (-62, -67), (-8, 44), (44, 17), (-91, -74), (-56, 18),
    (96, -19), (45, -67), (-28, 62), (94, 69), (48, 52),
    (-11, 64), (-95, -57), (-2, 79), (34, 40), (-5, 24),
    (-35, -50), (-40, 72), (-25, -4), (-75, -98), (6, 98),
    (-87, -37), (-63, 99), (-96, 86), (28, 65), (-87, 26),
    (53, -2), (-98, 7), (69, -71), (18, 41), (-84, 51),
    (-80, -10), (50, 39), (13, -89), (4, 35), (31, 95),
    (84, -50), (86, -82), (32, -21), (-36, -22), (34, -77),
    (-77, -78), (-92, -2), (72, -54), (88, -29), (1, -14),
    (-82, 97), (-16, -70), (-19, 96), (-41, 41), (-24, -87)
]

print("Trying different approaches to find a solution under 1400...")

best_path = None
best_distance = float('inf')

# Try multiple starting approaches
approaches = [
    ("Standard nearest neighbor", None),
    ("Start from flower closest to origin", None),
    ("Try multiple random starting flowers", "random")
]

for approach_name, method in approaches:
    print(f"\n{approach_name}:")
    
    if method == "random":
        # Try starting from different flowers
        for start_flower in range(min(10, len(coordinates))):
            path, dist = nearest_neighbor_from_start(coordinates, start_flower)
            if dist < best_distance:
                best_path = path[:]
                best_distance = dist
                print(f"  Starting from flower {start_flower + 1}: {dist:.2f}")
    else:
        if approach_name.startswith("Start from flower"):
            # Find flower closest to origin
            min_dist = float('inf')
            closest_flower = 0
            for i, coord in enumerate(coordinates):
                d = distance((0, 0), coord)
                if d < min_dist:
                    min_dist = d
                    closest_flower = i
            path, dist = nearest_neighbor_from_start(coordinates, closest_flower)
        else:
            path, dist = nearest_neighbor_from_start(coordinates)
        
        if dist < best_distance:
            best_path = path[:]
            best_distance = dist
        
        print(f"  Distance: {dist:.2f}")

print(f"\nBest initial solution: {best_distance:.2f}")

# Apply 2-opt improvement
if best_path:
    print("\nApplying 2-opt improvements...")
    improved_path, improved_distance = two_opt_improve(best_path, coordinates)
    
    print(f"After 2-opt: {improved_distance:.2f}")
    
    if improved_distance < best_distance:
        best_path = improved_path
        best_distance = improved_distance

print(f"\nFinal solution:")
print("Path (beehive and flower indices):")
print(" ".join(map(str, best_path)))
print(f"\nTotal distance: {best_distance:.2f}")
print(f"Valid solution: {best_distance < 1400}")

# Verify the path
if best_path:
    flowers_visited = set(best_path[1:-1])
    print(f"Flowers visited: {len(flowers_visited)} out of {len(coordinates)}")
    print(f"All flowers visited exactly once: {len(flowers_visited) == len(coordinates)}")

```

Send the outputted path to the server for the flag.

```
echo "0 45 38 26 3 32 10 14 29 34 15 2 11 8 17 49 5 1 41 19 4 12 21 31 42 27 25 30 23 46 22 48 13 20 35 24 9 6 44 36 43 37 28 7 40 33 50 47 16 39 18 0" | nc 0.cloud.chals.io 13055
```

![](Images/Pasted%20image%2020250531102420.png)
