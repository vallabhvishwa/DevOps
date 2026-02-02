# DevOps Engineer's Complete Reference Guide
# Part 18: Data Structures & Algorithms for DevOps/SRE Interviews

---

## Table of Contents

1. [Why DSA for DevOps](#1-why-dsa-for-devops)
2. [Essential Data Structures](#2-essential-data-structures)
3. [Essential Algorithms](#3-essential-algorithms)
4. [Common Interview Patterns](#4-common-interview-patterns)
5. [DevOps-Specific Problems](#5-devops-specific-problems)
6. [LeetCode Problem Guide](#6-leetcode-problem-guide)
7. [Interview Tips](#7-interview-tips)

---

## 1. Why DSA for DevOps

```
FAANG DevOps/SRE Interview Structure:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Typical Interview Loop (5-6 rounds):                           │
│                                                                 │
│  1. CODING (1-2 rounds)                                         │
│     ├── LeetCode Easy/Medium problems                           │
│     ├── Scripting problems (Python/Go)                          │
│     └── Focus: Problem-solving, clean code                      │
│                                                                 │
│  2. SYSTEM DESIGN (1-2 rounds)                                  │
│     ├── Design a CI/CD system                                   │
│     ├── Design a logging pipeline                               │
│     └── Focus: Scalability, reliability                         │
│                                                                 │
│  3. TECHNICAL DEEP DIVE (1-2 rounds)                            │
│     ├── Linux, Kubernetes, networking                           │
│     ├── Past experience deep dive                               │
│     └── Focus: Depth of knowledge                               │
│                                                                 │
│  4. BEHAVIORAL (1 round)                                        │
│     ├── Leadership principles                                   │
│     └── Focus: Communication, conflict resolution               │
│                                                                 │
│  Coding Round Expectations:                                     │
│  ├── Solve 1-2 problems in 45-60 minutes                        │
│  ├── Difficulty: Easy to Medium (rarely Hard)                   │
│  ├── Languages: Python, Go, Java acceptable                     │
│  └── Focus: Working solution + clean code + edge cases          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Essential Data Structures

### 2.1 Arrays and Strings

```python
# Arrays - Most common in interviews

# Basic operations
arr = [1, 2, 3, 4, 5]
arr.append(6)           # O(1) - Add to end
arr.pop()               # O(1) - Remove from end
arr.insert(0, 0)        # O(n) - Insert at index
arr.remove(3)           # O(n) - Remove by value
len(arr)                # O(1) - Get length

# Two Pointer Technique
def two_sum_sorted(arr, target):
    """Find two numbers that sum to target in sorted array."""
    left, right = 0, len(arr) - 1
    while left < right:
        current_sum = arr[left] + arr[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    return []

# Sliding Window
def max_sum_subarray(arr, k):
    """Find max sum of subarray of size k."""
    if len(arr) < k:
        return 0
    
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

# String manipulation
s = "hello world"
s.split()               # ['hello', 'world']
s.replace('l', 'x')     # 'hexxo worxd'
s[::-1]                 # 'dlrow olleh' (reverse)
''.join(['a', 'b'])     # 'ab'
s.startswith('hello')   # True
s.find('world')         # 6 (index)
```

### 2.2 Hash Maps (Dictionaries)

```python
# Hash Map - O(1) average lookup, insert, delete

# Two Sum (Classic Interview Problem)
def two_sum(nums, target):
    """Find indices of two numbers that sum to target."""
    seen = {}  # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

# Frequency Counter
def find_duplicates(arr):
    """Find all duplicate elements."""
    freq = {}
    duplicates = []
    for num in arr:
        freq[num] = freq.get(num, 0) + 1
        if freq[num] == 2:
            duplicates.append(num)
    return duplicates

# Group Anagrams
def group_anagrams(strs):
    """Group strings that are anagrams of each other."""
    groups = {}
    for s in strs:
        key = ''.join(sorted(s))
        if key not in groups:
            groups[key] = []
        groups[key].append(s)
    return list(groups.values())

# Counter (Python built-in)
from collections import Counter
counts = Counter(['a', 'b', 'a', 'c', 'a'])
# Counter({'a': 3, 'b': 1, 'c': 1})
counts.most_common(2)  # [('a', 3), ('b', 1)]
```

### 2.3 Sets

```python
# Set - O(1) lookup, no duplicates

# Remove duplicates
def unique_elements(arr):
    return list(set(arr))

# Intersection of two arrays
def intersection(arr1, arr2):
    return list(set(arr1) & set(arr2))

# Union
def union(arr1, arr2):
    return list(set(arr1) | set(arr2))

# Find missing number
def find_missing(nums, n):
    """Find missing number from 1 to n."""
    expected = set(range(1, n + 1))
    actual = set(nums)
    return list(expected - actual)[0]
```

### 2.4 Stacks and Queues

```python
# Stack - LIFO (Last In, First Out)
stack = []
stack.append(1)  # Push
stack.pop()      # Pop
stack[-1]        # Peek (top element)

# Valid Parentheses (Classic Problem)
def is_valid_parentheses(s):
    """Check if parentheses are balanced."""
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            stack.append(char)
    
    return len(stack) == 0

# Queue - FIFO (First In, First Out)
from collections import deque
queue = deque()
queue.append(1)     # Enqueue (right)
queue.popleft()     # Dequeue (left)
queue[0]            # Peek (front)

# Monotonic Stack (for next greater element)
def next_greater_element(nums):
    """Find next greater element for each position."""
    result = [-1] * len(nums)
    stack = []  # Store indices
    
    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            result[idx] = num
        stack.append(i)
    
    return result
```

### 2.5 Linked Lists

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Reverse Linked List
def reverse_list(head):
    """Reverse a singly linked list."""
    prev = None
    curr = head
    
    while curr:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    
    return prev

# Detect Cycle (Floyd's Algorithm)
def has_cycle(head):
    """Detect if linked list has a cycle."""
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    
    return False

# Merge Two Sorted Lists
def merge_two_lists(l1, l2):
    """Merge two sorted linked lists."""
    dummy = ListNode(0)
    curr = dummy
    
    while l1 and l2:
        if l1.val < l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    
    curr.next = l1 or l2
    return dummy.next
```

### 2.6 Trees

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Tree Traversals
def inorder(root):
    """Left -> Root -> Right"""
    if not root:
        return []
    return inorder(root.left) + [root.val] + inorder(root.right)

def preorder(root):
    """Root -> Left -> Right"""
    if not root:
        return []
    return [root.val] + preorder(root.left) + preorder(root.right)

def postorder(root):
    """Left -> Right -> Root"""
    if not root:
        return []
    return postorder(root.left) + postorder(root.right) + [root.val]

# Level Order (BFS)
def level_order(root):
    """Breadth-first traversal."""
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    
    return result

# Max Depth of Binary Tree
def max_depth(root):
    """Find maximum depth of tree."""
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

# Validate Binary Search Tree
def is_valid_bst(root, min_val=float('-inf'), max_val=float('inf')):
    """Check if tree is valid BST."""
    if not root:
        return True
    
    if root.val <= min_val or root.val >= max_val:
        return False
    
    return (is_valid_bst(root.left, min_val, root.val) and
            is_valid_bst(root.right, root.val, max_val))
```

### 2.7 Graphs

```python
from collections import deque, defaultdict

# Graph representation
# Adjacency List (most common)
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D', 'E'],
    'C': ['A', 'F'],
    'D': ['B'],
    'E': ['B', 'F'],
    'F': ['C', 'E']
}

# BFS - Breadth First Search
def bfs(graph, start):
    """Visit all nodes level by level."""
    visited = set()
    queue = deque([start])
    result = []
    
    while queue:
        node = queue.popleft()
        if node not in visited:
            visited.add(node)
            result.append(node)
            for neighbor in graph.get(node, []):
                if neighbor not in visited:
                    queue.append(neighbor)
    
    return result

# DFS - Depth First Search
def dfs(graph, start, visited=None):
    """Visit all nodes depth-first."""
    if visited is None:
        visited = set()
    
    visited.add(start)
    result = [start]
    
    for neighbor in graph.get(start, []):
        if neighbor not in visited:
            result.extend(dfs(graph, neighbor, visited))
    
    return result

# Shortest Path (BFS for unweighted)
def shortest_path(graph, start, end):
    """Find shortest path between two nodes."""
    queue = deque([(start, [start])])
    visited = set([start])
    
    while queue:
        node, path = queue.popleft()
        if node == end:
            return path
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    
    return []  # No path found

# Detect Cycle in Directed Graph
def has_cycle_directed(graph):
    """Detect cycle using DFS with colors."""
    WHITE, GRAY, BLACK = 0, 1, 2
    color = defaultdict(int)
    
    def dfs(node):
        color[node] = GRAY
        for neighbor in graph.get(node, []):
            if color[neighbor] == GRAY:  # Back edge
                return True
            if color[neighbor] == WHITE and dfs(neighbor):
                return True
        color[node] = BLACK
        return False
    
    return any(dfs(node) for node in graph if color[node] == WHITE)

# Topological Sort (for DAG - Directed Acyclic Graph)
def topological_sort(graph):
    """Return nodes in topological order."""
    in_degree = defaultdict(int)
    for node in graph:
        for neighbor in graph[node]:
            in_degree[neighbor] += 1
    
    queue = deque([n for n in graph if in_degree[n] == 0])
    result = []
    
    while queue:
        node = queue.popleft()
        result.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    
    return result if len(result) == len(graph) else []  # Empty if cycle
```

### 2.8 Heaps (Priority Queues)

```python
import heapq

# Min Heap (default in Python)
min_heap = []
heapq.heappush(min_heap, 3)
heapq.heappush(min_heap, 1)
heapq.heappush(min_heap, 2)
smallest = heapq.heappop(min_heap)  # Returns 1

# Max Heap (negate values)
max_heap = []
heapq.heappush(max_heap, -3)
heapq.heappush(max_heap, -1)
largest = -heapq.heappop(max_heap)  # Returns 3

# Kth Largest Element
def find_kth_largest(nums, k):
    """Find kth largest element."""
    # Use min heap of size k
    heap = nums[:k]
    heapq.heapify(heap)
    
    for num in nums[k:]:
        if num > heap[0]:
            heapq.heapreplace(heap, num)
    
    return heap[0]

# Top K Frequent Elements
def top_k_frequent(nums, k):
    """Find k most frequent elements."""
    counts = Counter(nums)
    return [item for item, count in counts.most_common(k)]

# Merge K Sorted Lists
def merge_k_lists(lists):
    """Merge k sorted lists into one."""
    heap = []
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))
    
    result = []
    while heap:
        val, list_idx, elem_idx = heapq.heappop(heap)
        result.append(val)
        
        if elem_idx + 1 < len(lists[list_idx]):
            next_val = lists[list_idx][elem_idx + 1]
            heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))
    
    return result
```

---

## 3. Essential Algorithms

### 3.1 Binary Search

```python
def binary_search(arr, target):
    """Find target in sorted array. O(log n)"""
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

# Binary Search Variations

def find_first_occurrence(arr, target):
    """Find first occurrence of target."""
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            result = mid
            right = mid - 1  # Continue searching left
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return result

def search_rotated_array(nums, target):
    """Search in rotated sorted array."""
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        
        # Left half is sorted
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        # Right half is sorted
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1
```

### 3.2 Sorting Algorithms

```python
# Quick Sort - O(n log n) average, O(n²) worst
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quicksort(left) + middle + quicksort(right)

# Merge Sort - O(n log n) always
def mergesort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = mergesort(arr[:mid])
    right = mergesort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result

# Python built-in (Timsort - O(n log n))
arr.sort()              # In-place
sorted_arr = sorted(arr)  # Returns new list
sorted_arr = sorted(arr, key=lambda x: x[1])  # Custom key
sorted_arr = sorted(arr, reverse=True)  # Descending
```

### 3.3 Dynamic Programming

```python
# Fibonacci (Classic DP Example)
def fibonacci(n):
    """Calculate nth Fibonacci number."""
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    
    return dp[n]

# Space Optimized
def fibonacci_optimized(n):
    if n <= 1:
        return n
    
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr
    
    return curr

# Climbing Stairs
def climb_stairs(n):
    """Number of ways to climb n stairs (1 or 2 steps at a time)."""
    if n <= 2:
        return n
    
    prev, curr = 1, 2
    for _ in range(3, n + 1):
        prev, curr = curr, prev + curr
    
    return curr

# Coin Change
def coin_change(coins, amount):
    """Minimum coins to make amount."""
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for x in range(coin, amount + 1):
            dp[x] = min(dp[x], dp[x - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

# Longest Common Subsequence
def lcs(text1, text2):
    """Find length of longest common subsequence."""
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    return dp[m][n]
```

---

## 4. Common Interview Patterns

```
Pattern Recognition:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. TWO POINTERS                                                │
│     Use when: Sorted array, finding pairs, palindromes          │
│     Examples: Two Sum II, Container With Most Water             │
│                                                                 │
│  2. SLIDING WINDOW                                              │
│     Use when: Subarray/substring problems, fixed or variable    │
│     Examples: Max Sum Subarray, Longest Substring               │
│                                                                 │
│  3. HASH MAP                                                    │
│     Use when: Finding pairs, counting, grouping                 │
│     Examples: Two Sum, Group Anagrams                           │
│                                                                 │
│  4. BINARY SEARCH                                               │
│     Use when: Sorted data, finding threshold                    │
│     Examples: Search in Rotated Array, First Bad Version        │
│                                                                 │
│  5. BFS                                                         │
│     Use when: Shortest path, level-order traversal              │
│     Examples: Word Ladder, Binary Tree Level Order              │
│                                                                 │
│  6. DFS                                                         │
│     Use when: Exploring all paths, tree traversal               │
│     Examples: Path Sum, Number of Islands                       │
│                                                                 │
│  7. DYNAMIC PROGRAMMING                                         │
│     Use when: Optimal substructure, overlapping subproblems     │
│     Examples: Climbing Stairs, Coin Change                      │
│                                                                 │
│  8. HEAP                                                        │
│     Use when: Kth element, streaming data, merging              │
│     Examples: Kth Largest, Merge K Sorted Lists                 │
│                                                                 │
│  9. STACK                                                       │
│     Use when: Matching brackets, monotonic problems             │
│     Examples: Valid Parentheses, Daily Temperatures             │
│                                                                 │
│  10. UNION-FIND                                                 │
│      Use when: Connected components, grouping                   │
│      Examples: Number of Islands, Accounts Merge                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. DevOps-Specific Problems

### 5.1 Log Parsing

```python
def parse_log_file(logs):
    """
    Parse log entries and find error patterns.
    Input: ["2024-01-15 ERROR: Connection failed",
            "2024-01-15 INFO: Request processed",
            "2024-01-15 ERROR: Connection failed"]
    Output: Most common error
    """
    from collections import Counter
    
    errors = []
    for log in logs:
        if "ERROR" in log:
            # Extract error message
            error_msg = log.split("ERROR:")[-1].strip()
            errors.append(error_msg)
    
    if not errors:
        return None
    
    return Counter(errors).most_common(1)[0][0]

def find_log_time_range(logs, start_time, end_time):
    """Find logs within time range using binary search."""
    # Assume logs are sorted by timestamp
    def parse_time(log):
        return log.split()[0] + " " + log.split()[1]
    
    # Binary search for start
    left, right = 0, len(logs) - 1
    start_idx = len(logs)
    while left <= right:
        mid = (left + right) // 2
        if parse_time(logs[mid]) >= start_time:
            start_idx = mid
            right = mid - 1
        else:
            left = mid + 1
    
    # Binary search for end
    left, right = 0, len(logs) - 1
    end_idx = -1
    while left <= right:
        mid = (left + right) // 2
        if parse_time(logs[mid]) <= end_time:
            end_idx = mid
            left = mid + 1
        else:
            right = mid - 1
    
    return logs[start_idx:end_idx + 1]
```

### 5.2 Resource Scheduling

```python
def schedule_jobs(jobs, max_concurrent):
    """
    Schedule jobs with dependencies.
    Input: jobs = [(job_id, duration, dependencies), ...]
    """
    from collections import defaultdict, deque
    
    graph = defaultdict(list)
    in_degree = defaultdict(int)
    duration = {}
    
    for job_id, dur, deps in jobs:
        duration[job_id] = dur
        for dep in deps:
            graph[dep].append(job_id)
            in_degree[job_id] += 1
    
    # Topological sort with timing
    queue = deque()
    finish_time = {}
    
    for job_id, _, _ in jobs:
        if in_degree[job_id] == 0:
            queue.append(job_id)
            finish_time[job_id] = duration[job_id]
    
    while queue:
        job = queue.popleft()
        for next_job in graph[job]:
            in_degree[next_job] -= 1
            if in_degree[next_job] == 0:
                queue.append(next_job)
                finish_time[next_job] = finish_time[job] + duration[next_job]
    
    return max(finish_time.values()) if finish_time else 0
```

### 5.3 Rate Limiter

```python
from collections import deque
import time

class RateLimiter:
    """Sliding window rate limiter."""
    
    def __init__(self, max_requests, window_seconds):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = {}  # client_id -> deque of timestamps
    
    def is_allowed(self, client_id):
        current_time = time.time()
        
        if client_id not in self.requests:
            self.requests[client_id] = deque()
        
        # Remove old requests outside window
        while (self.requests[client_id] and 
               current_time - self.requests[client_id][0] > self.window_seconds):
            self.requests[client_id].popleft()
        
        if len(self.requests[client_id]) < self.max_requests:
            self.requests[client_id].append(current_time)
            return True
        
        return False
```

### 5.4 IP Address Manipulation

```python
def ip_to_int(ip):
    """Convert IP address to integer."""
    parts = ip.split('.')
    return (int(parts[0]) << 24) + (int(parts[1]) << 16) + \
           (int(parts[2]) << 8) + int(parts[3])

def int_to_ip(num):
    """Convert integer to IP address."""
    return f"{(num >> 24) & 255}.{(num >> 16) & 255}.{(num >> 8) & 255}.{num & 255}"

def is_ip_in_cidr(ip, cidr):
    """Check if IP is in CIDR range."""
    network, prefix = cidr.split('/')
    prefix = int(prefix)
    
    ip_int = ip_to_int(ip)
    network_int = ip_to_int(network)
    
    mask = (0xFFFFFFFF << (32 - prefix)) & 0xFFFFFFFF
    
    return (ip_int & mask) == (network_int & mask)

def merge_ip_ranges(ranges):
    """Merge overlapping IP ranges."""
    if not ranges:
        return []
    
    # Convert to integers and sort
    intervals = [(ip_to_int(start), ip_to_int(end)) for start, end in ranges]
    intervals.sort()
    
    merged = [intervals[0]]
    
    for start, end in intervals[1:]:
        if start <= merged[-1][1] + 1:
            merged[-1] = (merged[-1][0], max(merged[-1][1], end))
        else:
            merged.append((start, end))
    
    return [(int_to_ip(s), int_to_ip(e)) for s, e in merged]
```

---

## 6. LeetCode Problem Guide

### 6.1 Must-Do Problems (50 Essential)

```
ARRAYS & STRINGS (15):
├── Two Sum (Easy) ⭐
├── Best Time to Buy and Sell Stock (Easy) ⭐
├── Contains Duplicate (Easy)
├── Maximum Subarray (Medium) ⭐
├── Product of Array Except Self (Medium)
├── 3Sum (Medium)
├── Container With Most Water (Medium)
├── Valid Anagram (Easy)
├── Group Anagrams (Medium)
├── Longest Substring Without Repeating Characters (Medium) ⭐
├── Longest Palindromic Substring (Medium)
├── Valid Palindrome (Easy)
├── String to Integer (Medium)
├── Merge Intervals (Medium) ⭐
└── Meeting Rooms II (Medium)

LINKED LISTS (5):
├── Reverse Linked List (Easy) ⭐
├── Merge Two Sorted Lists (Easy) ⭐
├── Linked List Cycle (Easy) ⭐
├── Remove Nth Node From End (Medium)
└── Reorder List (Medium)

TREES (10):
├── Maximum Depth of Binary Tree (Easy) ⭐
├── Validate Binary Search Tree (Medium) ⭐
├── Binary Tree Level Order Traversal (Medium) ⭐
├── Lowest Common Ancestor (Medium)
├── Serialize and Deserialize Binary Tree (Hard)
├── Invert Binary Tree (Easy)
├── Same Tree (Easy)
├── Subtree of Another Tree (Easy)
├── Construct Binary Tree from Preorder and Inorder (Medium)
└── Kth Smallest Element in BST (Medium)

GRAPHS (5):
├── Number of Islands (Medium) ⭐
├── Clone Graph (Medium)
├── Course Schedule (Medium) ⭐
├── Pacific Atlantic Water Flow (Medium)
└── Graph Valid Tree (Medium)

DYNAMIC PROGRAMMING (10):
├── Climbing Stairs (Easy) ⭐
├── House Robber (Medium) ⭐
├── Coin Change (Medium) ⭐
├── Longest Increasing Subsequence (Medium)
├── Word Break (Medium)
├── Unique Paths (Medium)
├── Jump Game (Medium)
├── Decode Ways (Medium)
├── Maximum Product Subarray (Medium)
└── Longest Common Subsequence (Medium)

MISC (5):
├── Implement Trie (Medium)
├── Design Add and Search Words (Medium)
├── Top K Frequent Elements (Medium) ⭐
├── Find Median from Data Stream (Hard)
└── LRU Cache (Medium) ⭐

⭐ = High priority for DevOps/SRE interviews
```

### 6.2 Study Schedule

```
Week 1-2: Arrays & Strings (15 problems)
├── Day 1-2: Two pointers, sliding window
├── Day 3-4: Hash maps, frequency counting
├── Day 5-6: Intervals, string manipulation
└── Day 7: Review and practice

Week 3: Linked Lists & Stacks (8 problems)
├── Day 1-2: Linked list basics
├── Day 3-4: Stack problems
└── Day 5-7: Combined practice

Week 4: Trees & Graphs (15 problems)
├── Day 1-2: Tree traversals, DFS
├── Day 3-4: BFS, level order
├── Day 5-6: Graph traversal
└── Day 7: Review

Week 5-6: Dynamic Programming (10 problems)
├── Day 1-3: 1D DP problems
├── Day 4-6: 2D DP problems
└── Day 7: Review

Week 7-8: Mixed Practice
├── Mock interviews
├── Timed problem solving
└── Review weak areas
```

---

## 7. Interview Tips

```
Coding Interview Strategy:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. UNDERSTAND (2-3 min)                                        │
│     ├── Repeat the problem back                                 │
│     ├── Ask clarifying questions                                │
│     ├── Discuss edge cases                                      │
│     └── Confirm input/output format                             │
│                                                                 │
│  2. PLAN (3-5 min)                                              │
│     ├── Think out loud                                          │
│     ├── Discuss multiple approaches                             │
│     ├── Analyze time/space complexity                           │
│     └── Choose best approach for constraints                    │
│                                                                 │
│  3. CODE (15-20 min)                                            │
│     ├── Write clean, readable code                              │
│     ├── Use meaningful variable names                           │
│     ├── Handle edge cases                                       │
│     └── Comment complex logic                                   │
│                                                                 │
│  4. TEST (5-10 min)                                             │
│     ├── Trace through with example                              │
│     ├── Test edge cases                                         │
│     ├── Fix bugs calmly                                         │
│     └── Discuss potential improvements                          │
│                                                                 │
│  Common Mistakes to Avoid:                                      │
│  ├── Jumping to code without understanding                      │
│  ├── Not asking clarifying questions                            │
│  ├── Silent coding (always explain)                             │
│  ├── Ignoring edge cases                                        │
│  └── Giving up when stuck                                       │
│                                                                 │
│  What to Say When Stuck:                                        │
│  ├── "Let me think about this differently..."                   │
│  ├── "What if I tried a different data structure?"              │
│  ├── "Can you give me a hint about the approach?"               │
│  └── "Let me trace through a smaller example..."                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

For FAANG DevOps/SRE coding interviews:

1. **Focus Areas**: Arrays, Strings, Hash Maps, Trees, Graphs, basic DP
2. **Difficulty**: Mostly Easy to Medium (rarely Hard)
3. **Languages**: Python recommended (concise, readable)
4. **Practice**: 50-100 problems, 1-2 months preparation
5. **Pattern Recognition**: Learn to identify problem types quickly
6. **Communication**: Always explain your thought process

Practice daily, even 30 minutes helps. Use LeetCode, HackerRank, or similar platforms.

---

**Next Part**: [Part 19: Message Queues & Event Streaming](./DevOps-Complete-Reference-Guide-Part19-Messaging.md)
