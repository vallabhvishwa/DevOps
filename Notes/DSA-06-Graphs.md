# Complete DSA Guide - Crack Any Coding Interview
# Part 6: Graphs - BFS & DFS

---

## Table of Contents

1. [Graph Fundamentals](#1-graph-fundamentals)
2. [Graph Representations](#2-graph-representations)
3. [BFS (Breadth-First Search)](#3-bfs-breadth-first-search)
4. [DFS (Depth-First Search)](#4-dfs-depth-first-search)
5. [Common Graph Problems](#5-common-graph-problems)
6. [Advanced Graph Algorithms](#6-advanced-graph-algorithms)
7. [Practice Problems](#7-practice-problems)

---

## 1. Graph Fundamentals

```
Graph Terminology:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Graph G = (V, E)                                               │
│  V = Vertices (nodes)                                           │
│  E = Edges (connections)                                        │
│                                                                 │
│  UNDIRECTED GRAPH:        DIRECTED GRAPH (Digraph):             │
│    A ─── B                  A ───► B                            │
│    │     │                  │      ↓                            │
│    C ─── D                  C ◄─── D                            │
│                                                                 │
│  WEIGHTED GRAPH:          UNWEIGHTED GRAPH:                     │
│    A ─5─ B                  A ─── B                             │
│    │     │                  │     │                             │
│   10     2                  │     │                             │
│    │     │                  │     │                             │
│    C ─3─ D                  C ─── D                             │
│                                                                 │
│  Terminology:                                                   │
│  ├── Degree: Number of edges connected to a vertex              │
│  ├── Path: Sequence of vertices connected by edges              │
│  ├── Cycle: Path that starts and ends at same vertex            │
│  ├── Connected: Path exists between every pair of vertices      │
│  ├── DAG: Directed Acyclic Graph (no cycles)                    │
│  └── Tree: Connected graph with no cycles                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Graph Representations

### 2.1 Adjacency List (Most Common)

```python
# Using dictionary
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D'],
    'C': ['A', 'D'],
    'D': ['B', 'C']
}

# Using defaultdict
from collections import defaultdict
graph = defaultdict(list)
graph['A'].append('B')
graph['B'].append('A')

# For weighted graphs: store (neighbor, weight)
weighted_graph = {
    'A': [('B', 5), ('C', 10)],
    'B': [('A', 5), ('D', 2)],
    'C': [('A', 10), ('D', 3)],
    'D': [('B', 2), ('C', 3)]
}

# Building from edge list
edges = [['A', 'B'], ['B', 'C'], ['C', 'D']]
graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)  # For undirected
```

### 2.2 Adjacency Matrix

```python
# For n vertices (0 to n-1)
n = 4
matrix = [[0] * n for _ in range(n)]

# Add edge between 0 and 1
matrix[0][1] = 1
matrix[1][0] = 1  # For undirected

# For weighted
matrix[0][1] = 5  # Weight instead of 1

# Check if edge exists: O(1)
has_edge = matrix[0][1] != 0

# Space: O(n²) - good for dense graphs
```

---

## 3. BFS (Breadth-First Search)

### 3.1 BFS Template

```python
from collections import deque

def bfs(graph, start):
    """
    BFS explores level by level.
    Uses queue (FIFO).
    
    Time: O(V + E), Space: O(V)
    """
    visited = set([start])
    queue = deque([start])
    
    while queue:
        node = queue.popleft()
        print(node)  # Process node
        
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

### 3.2 Shortest Path (Unweighted)

```python
def shortest_path(graph, start, end):
    """
    BFS finds shortest path in unweighted graph.
    
    Time: O(V + E), Space: O(V)
    """
    if start == end:
        return [start]
    
    visited = set([start])
    queue = deque([(start, [start])])  # (node, path)
    
    while queue:
        node, path = queue.popleft()
        
        for neighbor in graph[node]:
            if neighbor == end:
                return path + [neighbor]
            
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    
    return []  # No path found
```

### 3.3 Level Order BFS

```python
def bfs_levels(graph, start):
    """BFS that tracks levels/distance."""
    visited = set([start])
    queue = deque([start])
    level = 0
    
    while queue:
        print(f"Level {level}:", end=" ")
        
        for _ in range(len(queue)):  # Process entire level
            node = queue.popleft()
            print(node, end=" ")
            
            for neighbor in graph[node]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)
        
        print()
        level += 1
```

---

## 4. DFS (Depth-First Search)

### 4.1 DFS Templates

```python
def dfs_recursive(graph, start, visited=None):
    """
    DFS explores as deep as possible before backtracking.
    
    Time: O(V + E), Space: O(V)
    """
    if visited is None:
        visited = set()
    
    visited.add(start)
    print(start)  # Process node
    
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)


def dfs_iterative(graph, start):
    """DFS using explicit stack."""
    visited = set()
    stack = [start]
    
    while stack:
        node = stack.pop()
        
        if node not in visited:
            visited.add(node)
            print(node)  # Process node
            
            for neighbor in graph[node]:
                if neighbor not in visited:
                    stack.append(neighbor)
```

### 4.2 DFS for Paths

```python
def find_all_paths(graph, start, end):
    """Find all paths from start to end."""
    result = []
    
    def dfs(node, path):
        if node == end:
            result.append(path[:])
            return
        
        for neighbor in graph[node]:
            if neighbor not in path:  # Avoid cycles
                path.append(neighbor)
                dfs(neighbor, path)
                path.pop()  # Backtrack
    
    dfs(start, [start])
    return result
```

---

## 5. Common Graph Problems

### 5.1 Number of Islands

```python
"""
Problem: Count number of islands in 2D grid.
Input: grid = [
    ["1","1","0","0","0"],
    ["1","1","0","0","0"],
    ["0","0","1","0","0"],
    ["0","0","0","1","1"]
]
Output: 3
"""

def num_islands(grid):
    """
    DFS to explore and mark each island.
    
    Time: O(m * n), Space: O(m * n)
    """
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    count = 0
    
    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols:
            return
        if grid[r][c] != '1':
            return
        
        grid[r][c] = '0'  # Mark visited
        
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                dfs(r, c)
    
    return count
```

### 5.2 Clone Graph

```python
"""
Problem: Deep copy of a graph.
"""

def clone_graph(node):
    """
    DFS with hash map to track cloned nodes.
    
    Time: O(V + E), Space: O(V)
    """
    if not node:
        return None
    
    cloned = {}
    
    def dfs(original):
        if original in cloned:
            return cloned[original]
        
        copy = Node(original.val)
        cloned[original] = copy
        
        for neighbor in original.neighbors:
            copy.neighbors.append(dfs(neighbor))
        
        return copy
    
    return dfs(node)
```

### 5.3 Course Schedule (Cycle Detection)

```python
"""
Problem: Can you finish all courses given prerequisites?
Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: True
"""

def can_finish(numCourses, prerequisites):
    """
    Check if directed graph has a cycle (DFS with coloring).
    
    Time: O(V + E), Space: O(V)
    """
    graph = defaultdict(list)
    for course, prereq in prerequisites:
        graph[prereq].append(course)
    
    # 0 = unvisited, 1 = visiting, 2 = visited
    state = [0] * numCourses
    
    def has_cycle(course):
        if state[course] == 1:  # Currently visiting = cycle
            return True
        if state[course] == 2:  # Already fully visited
            return False
        
        state[course] = 1  # Mark visiting
        
        for next_course in graph[course]:
            if has_cycle(next_course):
                return True
        
        state[course] = 2  # Mark visited
        return False
    
    for course in range(numCourses):
        if has_cycle(course):
            return False
    
    return True
```

### 5.4 Topological Sort

```python
"""
Problem: Order courses so prerequisites come first.
"""

def find_order(numCourses, prerequisites):
    """
    Topological sort using Kahn's algorithm (BFS).
    
    Time: O(V + E), Space: O(V)
    """
    graph = defaultdict(list)
    in_degree = [0] * numCourses
    
    for course, prereq in prerequisites:
        graph[prereq].append(course)
        in_degree[course] += 1
    
    # Start with courses that have no prerequisites
    queue = deque([c for c in range(numCourses) if in_degree[c] == 0])
    order = []
    
    while queue:
        course = queue.popleft()
        order.append(course)
        
        for next_course in graph[course]:
            in_degree[next_course] -= 1
            if in_degree[next_course] == 0:
                queue.append(next_course)
    
    return order if len(order) == numCourses else []


def topological_sort_dfs(numCourses, prerequisites):
    """Topological sort using DFS."""
    graph = defaultdict(list)
    for course, prereq in prerequisites:
        graph[prereq].append(course)
    
    visited = set()
    stack = []
    has_cycle = [False]
    
    def dfs(course, path):
        if course in path:  # Cycle
            has_cycle[0] = True
            return
        if course in visited:
            return
        
        path.add(course)
        for next_course in graph[course]:
            dfs(next_course, path)
        path.remove(course)
        
        visited.add(course)
        stack.append(course)
    
    for course in range(numCourses):
        dfs(course, set())
    
    return [] if has_cycle[0] else stack[::-1]
```

### 5.5 Word Ladder

```python
"""
Problem: Transform word to another, one letter at a time.
Input: beginWord = "hit", endWord = "cog"
       wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5 (hit -> hot -> dot -> dog -> cog)
"""

def ladder_length(beginWord, endWord, wordList):
    """
    BFS for shortest transformation sequence.
    
    Time: O(M² * N) where M = word length, N = word list size
    """
    if endWord not in wordList:
        return 0
    
    word_set = set(wordList)
    queue = deque([(beginWord, 1)])
    visited = set([beginWord])
    
    while queue:
        word, steps = queue.popleft()
        
        if word == endWord:
            return steps
        
        # Try changing each character
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                new_word = word[:i] + c + word[i+1:]
                
                if new_word in word_set and new_word not in visited:
                    visited.add(new_word)
                    queue.append((new_word, steps + 1))
    
    return 0
```

---

## 6. Advanced Graph Algorithms

### 6.1 Dijkstra's Algorithm (Weighted Shortest Path)

```python
"""
Find shortest path in weighted graph (no negative weights).
"""

import heapq

def dijkstra(graph, start):
    """
    Time: O((V + E) log V), Space: O(V)
    """
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    
    pq = [(0, start)]  # (distance, node)
    
    while pq:
        dist, node = heapq.heappop(pq)
        
        if dist > distances[node]:
            continue  # Already found shorter path
        
        for neighbor, weight in graph[node]:
            new_dist = dist + weight
            
            if new_dist < distances[neighbor]:
                distances[neighbor] = new_dist
                heapq.heappush(pq, (new_dist, neighbor))
    
    return distances
```

### 6.2 Union-Find (Disjoint Set)

```python
"""
Track connected components efficiently.
"""

class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.count = n  # Number of components
    
    def find(self, x):
        """Find root with path compression."""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """Union by rank."""
        px, py = self.find(x), self.find(y)
        
        if px == py:
            return False  # Already connected
        
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        
        self.count -= 1
        return True


# Number of Connected Components
def count_components(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.count
```

---

## 7. Practice Problems

```
BFS:
├── 200. Number of Islands (Medium) ⭐
├── 994. Rotting Oranges (Medium)
├── 127. Word Ladder (Hard)
├── 752. Open the Lock (Medium)
├── 1091. Shortest Path in Binary Matrix (Medium)

DFS:
├── 200. Number of Islands (Medium) ⭐
├── 133. Clone Graph (Medium)
├── 695. Max Area of Island (Medium)
├── 417. Pacific Atlantic Water Flow (Medium)
├── 130. Surrounded Regions (Medium)

TOPOLOGICAL SORT:
├── 207. Course Schedule (Medium) ⭐
├── 210. Course Schedule II (Medium) ⭐
├── 269. Alien Dictionary (Hard)

UNION-FIND:
├── 547. Number of Provinces (Medium)
├── 684. Redundant Connection (Medium)
├── 323. Number of Connected Components (Medium)

SHORTEST PATH:
├── 743. Network Delay Time (Medium)
├── 787. Cheapest Flights Within K Stops (Medium)
├── 1514. Path with Maximum Probability (Medium)
```

---

## Summary

1. **BFS**: Level-by-level, shortest path in unweighted graphs
2. **DFS**: Go deep, backtrack, good for all paths/cycles
3. **Topological Sort**: Order dependencies (Kahn's or DFS)
4. **Union-Find**: Connected components efficiently
5. **Dijkstra**: Shortest path in weighted graphs

---

**Next Part**: [Part 7: Dynamic Programming](./DSA-Complete-Guide-Part7-DynamicProgramming.md)
