# About
A Volnayan breach has reached deep into your network, with lateral movement already underway. As Orion “Circuitbreaker” Zhao, you must deploy a firewall to isolate the threat — but the honeypot must remain connected for surveillance. Choose the cut point carefully: the fewer nodes exposed, the better your containment.



```
# take in the number
n = int(input())

# Build the graph from input
edges = []
graph = [[] for _ in range(n + 1)]

for _ in range(n - 1):
    line = input().strip()
    a, b = map(int, line.split(' - '))
    edges.append((a, b))
    graph[a].append(b)
    graph[b].append(a)

honeypot = int(input())

# calculate answer
# We need to find the minimum number of nodes reachable from root (node 1)
# after placing one firewall (removing one edge), while keeping honeypot reachable

# First, find the path from root (1) to honeypot
def find_path_to_honeypot(start, target, graph, n):
    parent = [-1] * (n + 1)
    visited = [False] * (n + 1)
    queue = [start]
    visited[start] = True
    
    while queue:
        node = queue.pop(0)
        if node == target:
            break
        
        for neighbor in graph[node]:
            if not visited[neighbor]:
                visited[neighbor] = True
                parent[neighbor] = node
                queue.append(neighbor)
    
    # Reconstruct path
    path = []
    current = target
    while current != -1:
        path.append(current)
        current = parent[current]
    
    return path[::-1]

# Get path from root to honeypot
honeypot_path = find_path_to_honeypot(1, honeypot, graph, n)
honeypot_path_edges = set()

for i in range(len(honeypot_path) - 1):
    a, b = honeypot_path[i], honeypot_path[i + 1]
    honeypot_path_edges.add((min(a, b), max(a, b)))

# Calculate subtree sizes using DFS from root
subtree_sizes = [0] * (n + 1)

def dfs_subtree_sizes(node, parent):
    subtree_sizes[node] = 1
    for neighbor in graph[node]:
        if neighbor != parent:
            dfs_subtree_sizes(neighbor, node)
            subtree_sizes[node] += subtree_sizes[neighbor]

dfs_subtree_sizes(1, -1)

# Find the maximum number of nodes we can disconnect
max_disconnected = 0

# Try removing each edge
for a, b in edges:
    edge = (min(a, b), max(a, b))
    
    # Skip if this edge is on the path to honeypot (can't remove it)
    if edge in honeypot_path_edges:
        continue
    
    # Determine which component gets disconnected
    # The component with smaller subtree size gets disconnected
    if subtree_sizes[a] < subtree_sizes[b]:
        # a's subtree is smaller, it gets disconnected
        disconnected = subtree_sizes[a]
    else:
        # b's subtree is smaller, it gets disconnected  
        disconnected = subtree_sizes[b]
    
    max_disconnected = max(max_disconnected, disconnected)

# The answer is total nodes minus maximum we can disconnect
answer = n - max_disconnected

# print answer
print(answer)
```