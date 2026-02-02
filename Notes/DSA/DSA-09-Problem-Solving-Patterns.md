# Complete DSA Guide - Crack Any Coding Interview
# Part 9: Problem-Solving Patterns & Interview Strategy

---

## Table of Contents

1. [Pattern Recognition](#1-pattern-recognition)
2. [Problem-Solving Framework](#2-problem-solving-framework)
3. [Common Mistakes to Avoid](#3-common-mistakes-to-avoid)
4. [Time Management](#4-time-management)
5. [Backtracking](#5-backtracking)
6. [Bit Manipulation](#6-bit-manipulation)
7. [Study Plan](#7-study-plan)

---

## 1. Pattern Recognition

### 1.1 Pattern Cheat Sheet

```
PATTERN RECOGNITION GUIDE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  KEYWORD → PATTERN                                              │
│                                                                 │
│  "Sorted array"                    → Binary Search              │
│  "Find pair/triplet summing to X"  → Two Pointers or Hash Map   │
│  "Subarray with property X"        → Sliding Window             │
│  "Substring with property X"       → Sliding Window             │
│  "Maximum/minimum subarray"        → DP or Kadane's             │
│  "Contiguous elements"             → Sliding Window             │
│  "Find duplicate"                  → Hash Set or Fast/Slow      │
│  "In-place modification"           → Two Pointers               │
│                                                                 │
│  "Level by level"                  → BFS                        │
│  "Shortest path (unweighted)"      → BFS                        │
│  "All paths/combinations"          → DFS/Backtracking           │
│  "Connected components"            → DFS/BFS or Union-Find      │
│  "Cycle detection"                 → DFS with colors            │
│  "Topological order"               → Kahn's BFS or DFS          │
│                                                                 │
│  "K largest/smallest"              → Heap                       │
│  "Merge K sorted"                  → Heap                       │
│  "Median of stream"                → Two Heaps                  │
│  "Next greater/smaller"            → Monotonic Stack            │
│  "Prefix matching"                 → Trie                       │
│                                                                 │
│  "Optimal way to..."               → DP                         │
│  "Count number of ways"            → DP                         │
│  "Can you reach..."                → DP or BFS                  │
│  "Minimum/Maximum cost"            → DP                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Data Structure Selection

```
Choosing the Right Data Structure:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  NEED                          → USE                            │
│                                                                 │
│  Fast lookup by key            → Hash Map                       │
│  Fast existence check          → Hash Set                       │
│  Ordered data + fast search    → BST or sorted array + binary   │
│  Get min/max quickly           → Heap                           │
│  FIFO processing               → Queue                          │
│  LIFO/matching/undo            → Stack                          │
│  Prefix operations             → Trie                           │
│  Range queries                 → Segment Tree                   │
│  Connected components          → Union-Find                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Problem-Solving Framework

### 2.1 The 4-Step Framework

```
STEP 1: UNDERSTAND (2-3 minutes)
┌─────────────────────────────────────────────────────────────────┐
│  □ Repeat the problem in your own words                         │
│  □ Ask clarifying questions:                                    │
│    - Input format and constraints?                              │
│    - Can input be empty? Negative? Duplicates?                  │
│    - What to return if no solution?                             │
│  □ Work through examples by hand                                │
│  □ Identify edge cases                                          │
└─────────────────────────────────────────────────────────────────┘

STEP 2: PLAN (3-5 minutes)
┌─────────────────────────────────────────────────────────────────┐
│  □ Identify the pattern/technique                               │
│  □ Think of multiple approaches                                 │
│  □ Analyze time/space complexity of each                        │
│  □ Choose best approach based on constraints                    │
│  □ Outline the solution in pseudocode                           │
└─────────────────────────────────────────────────────────────────┘

STEP 3: CODE (15-20 minutes)
┌─────────────────────────────────────────────────────────────────┐
│  □ Write clean, readable code                                   │
│  □ Use meaningful variable names                                │
│  □ Think out loud as you code                                   │
│  □ Handle edge cases                                            │
│  □ Don't optimize prematurely                                   │
└─────────────────────────────────────────────────────────────────┘

STEP 4: TEST (5-10 minutes)
┌─────────────────────────────────────────────────────────────────┐
│  □ Trace through with given example                             │
│  □ Test edge cases (empty, single element, etc.)                │
│  □ Look for off-by-one errors                                   │
│  □ Verify return values                                         │
│  □ Discuss optimizations if time allows                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 When You're Stuck

```python
"""
Strategies when stuck:
"""

# 1. TRY A SIMPLER VERSION
# Can you solve for n=1, n=2?
# Can you solve without constraints?

# 2. WORK BACKWARDS
# Start from the answer, how would you get there?

# 3. DRAW IT OUT
# Visualize the problem with diagrams

# 4. USE DIFFERENT DATA STRUCTURE
# "Can a hash map help? A heap? A stack?"

# 5. BREAK INTO SUBPROBLEMS
# Divide and conquer approach

# 6. THINK OF RELATED PROBLEMS
# "This reminds me of..."

# 7. ASK FOR A HINT
# Better than being silent for 10 minutes
```

---

## 3. Common Mistakes to Avoid

```
TOP 10 INTERVIEW MISTAKES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. ❌ Jumping to code without understanding                    │
│     ✓ Spend 2-3 min understanding and asking questions          │
│                                                                 │
│  2. ❌ Coding in silence                                        │
│     ✓ Always explain your thought process                       │
│                                                                 │
│  3. ❌ Not asking clarifying questions                          │
│     ✓ Clarify inputs, outputs, edge cases                       │
│                                                                 │
│  4. ❌ Ignoring edge cases                                      │
│     ✓ Handle empty input, single element, etc.                  │
│                                                                 │
│  5. ❌ Off-by-one errors                                        │
│     ✓ Double-check loop bounds and indices                      │
│                                                                 │
│  6. ❌ Not testing your code                                    │
│     ✓ Trace through with examples                               │
│                                                                 │
│  7. ❌ Giving up when stuck                                     │
│     ✓ Ask for hints, try different approaches                   │
│                                                                 │
│  8. ❌ Premature optimization                                   │
│     ✓ Get working solution first, then optimize                 │
│                                                                 │
│  9. ❌ Poor variable names                                      │
│     ✓ Use descriptive names (left, right not l, r)              │
│                                                                 │
│  10. ❌ Not considering time/space complexity                   │
│      ✓ Always state complexity of your solution                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Time Management

```
45-Minute Interview Timeline:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  0:00 - 0:03  │ Understand problem, ask questions               │
│  0:03 - 0:08  │ Plan approach, discuss trade-offs               │
│  0:08 - 0:30  │ Code solution                                   │
│  0:30 - 0:40  │ Test and debug                                  │
│  0:40 - 0:45  │ Discuss optimizations, follow-ups               │
│                                                                 │
│  RED FLAGS:                                                     │
│  • Still understanding at 5 min → Ask for clarification         │
│  • Still planning at 10 min → Start with brute force            │
│  • Still coding at 35 min → Wrap up, explain remaining          │
│  • Stuck for 5 min → Ask for a hint                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Backtracking

### 5.1 Backtracking Template

```python
def backtrack(path, choices):
    """
    General backtracking template.
    """
    if is_solution(path):
        result.append(path[:])
        return
    
    for choice in choices:
        if is_valid(choice, path):
            path.append(choice)          # Make choice
            backtrack(path, new_choices) # Recurse
            path.pop()                   # Undo choice (backtrack)
```

### 5.2 Subsets

```python
"""
Problem: Generate all subsets.
Input: nums = [1, 2, 3]
Output: [[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]
"""

def subsets(nums):
    result = []
    
    def backtrack(start, path):
        result.append(path[:])
        
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    
    backtrack(0, [])
    return result
```

### 5.3 Permutations

```python
"""
Problem: Generate all permutations.
Input: nums = [1, 2, 3]
Output: [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
"""

def permutations(nums):
    result = []
    
    def backtrack(path, remaining):
        if not remaining:
            result.append(path[:])
            return
        
        for i in range(len(remaining)):
            path.append(remaining[i])
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()
    
    backtrack([], nums)
    return result
```

### 5.4 Combination Sum

```python
"""
Problem: Find combinations that sum to target (reuse allowed).
Input: candidates = [2, 3, 6, 7], target = 7
Output: [[2, 2, 3], [7]]
"""

def combination_sum(candidates, target):
    result = []
    
    def backtrack(start, path, remaining):
        if remaining == 0:
            result.append(path[:])
            return
        if remaining < 0:
            return
        
        for i in range(start, len(candidates)):
            path.append(candidates[i])
            backtrack(i, path, remaining - candidates[i])  # i, not i+1 for reuse
            path.pop()
    
    backtrack(0, [], target)
    return result
```

### 5.5 N-Queens

```python
"""
Problem: Place N queens on NxN board with no attacks.
"""

def solve_n_queens(n):
    result = []
    board = [['.'] * n for _ in range(n)]
    cols = set()
    diag1 = set()  # row - col
    diag2 = set()  # row + col
    
    def backtrack(row):
        if row == n:
            result.append([''.join(r) for r in board])
            return
        
        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue
            
            board[row][col] = 'Q'
            cols.add(col)
            diag1.add(row - col)
            diag2.add(row + col)
            
            backtrack(row + 1)
            
            board[row][col] = '.'
            cols.remove(col)
            diag1.remove(row - col)
            diag2.remove(row + col)
    
    backtrack(0)
    return result
```

---

## 6. Bit Manipulation

### 6.1 Bit Basics

```python
# Basic operations
x & y   # AND
x | y   # OR
x ^ y   # XOR (toggle bits)
~x      # NOT (invert all bits)
x << n  # Left shift (multiply by 2^n)
x >> n  # Right shift (divide by 2^n)

# Common tricks
x & 1           # Check if odd (last bit is 1)
x & (x - 1)     # Remove lowest set bit
x | (x + 1)     # Set lowest unset bit
x ^ x           # Always 0
x ^ 0           # Always x
x & -x          # Isolate lowest set bit

# Count set bits
bin(x).count('1')

# Check if power of 2
x > 0 and (x & (x - 1)) == 0
```

### 6.2 Single Number

```python
"""
Problem: Find the only non-duplicate in array.
Every element appears twice except one.
"""

def single_number(nums):
    """
    XOR all numbers. Duplicates cancel out.
    
    Time: O(n), Space: O(1)
    """
    result = 0
    for num in nums:
        result ^= num
    return result
```

### 6.3 Missing Number

```python
"""
Problem: Find missing number in [0, n].
Input: nums = [3, 0, 1]
Output: 2
"""

def missing_number(nums):
    """XOR index ^ value. Missing number remains."""
    result = len(nums)
    
    for i, num in enumerate(nums):
        result ^= i ^ num
    
    return result
```

---

## 7. Study Plan

### 7.1 8-Week Study Plan

```
WEEK 1-2: FOUNDATIONS
├── Arrays & Strings (15 problems)
├── Two Pointers
├── Sliding Window
└── Binary Search

WEEK 3: HASH TABLES
├── Hash Map problems (10)
├── Hash Set problems (5)
└── Two Sum variations

WEEK 4: LINKED LISTS & STACKS
├── Linked List (8 problems)
├── Stacks (7 problems)
└── Queues (5 problems)

WEEK 5: TREES
├── Tree Traversals
├── BST problems (10)
├── Tree construction

WEEK 6: GRAPHS
├── BFS (8 problems)
├── DFS (8 problems)
├── Topological Sort

WEEK 7: DYNAMIC PROGRAMMING
├── 1D DP (10 problems)
├── 2D DP (10 problems)
├── Common patterns

WEEK 8: ADVANCED & REVIEW
├── Heaps (5 problems)
├── Tries (3 problems)
├── Backtracking (5 problems)
├── Mock interviews
```

### 7.2 Must-Do Problems (75 Problems)

```
ARRAYS & STRINGS (15):
1. Two Sum
2. Best Time to Buy/Sell Stock
3. Contains Duplicate
4. Product of Array Except Self
5. Maximum Subarray
6. 3Sum
7. Container With Most Water
8. Merge Intervals
9. Valid Anagram
10. Group Anagrams
11. Longest Substring Without Repeating
12. Minimum Window Substring
13. Search in Rotated Sorted Array
14. Find Minimum in Rotated Sorted Array
15. Longest Palindromic Substring

LINKED LISTS (5):
16. Reverse Linked List
17. Merge Two Sorted Lists
18. Linked List Cycle
19. Remove Nth Node From End
20. Reorder List

STACKS & QUEUES (5):
21. Valid Parentheses
22. Min Stack
23. Daily Temperatures
24. Evaluate Reverse Polish Notation
25. Sliding Window Maximum

TREES (15):
26. Maximum Depth of Binary Tree
27. Same Tree
28. Invert Binary Tree
29. Binary Tree Level Order Traversal
30. Validate BST
31. Kth Smallest Element in BST
32. Lowest Common Ancestor
33. Binary Tree Right Side View
34. Construct Tree from Preorder/Inorder
35. Serialize and Deserialize
36. Diameter of Binary Tree
37. Binary Tree Maximum Path Sum
38. Subtree of Another Tree
39. Path Sum
40. Implement Trie

GRAPHS (10):
41. Number of Islands
42. Clone Graph
43. Course Schedule
44. Course Schedule II
45. Pacific Atlantic Water Flow
46. Word Ladder
47. Number of Connected Components
48. Graph Valid Tree
49. Alien Dictionary
50. Accounts Merge

HEAPS (5):
51. Kth Largest Element
52. Top K Frequent Elements
53. Find Median from Data Stream
54. Merge K Sorted Lists
55. Task Scheduler

DYNAMIC PROGRAMMING (15):
56. Climbing Stairs
57. House Robber
58. House Robber II
59. Coin Change
60. Longest Increasing Subsequence
61. Word Break
62. Unique Paths
63. Longest Common Subsequence
64. Edit Distance
65. Jump Game
66. Partition Equal Subset Sum
67. Decode Ways
68. Maximum Product Subarray
69. Target Sum
70. Palindromic Substrings

BACKTRACKING (5):
71. Subsets
72. Permutations
73. Combination Sum
74. Letter Combinations of Phone
75. Word Search
```

---

## Final Summary

```
KEY TAKEAWAYS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. PATTERNS > MEMORIZATION                                     │
│     Learn to recognize patterns, not memorize solutions         │
│                                                                 │
│  2. PRACTICE CONSISTENTLY                                       │
│     30-60 min daily > 8 hours once a week                       │
│                                                                 │
│  3. UNDERSTAND COMPLEXITY                                       │
│     Always know time & space complexity of your solution        │
│                                                                 │
│  4. COMMUNICATE CLEARLY                                         │
│     Interview is about problem-solving, not just coding         │
│                                                                 │
│  5. HANDLE EDGE CASES                                           │
│     Empty input, single element, duplicates                     │
│                                                                 │
│  6. START SIMPLE                                                │
│     Brute force first, then optimize                            │
│                                                                 │
│  7. PRACTICE UNDER PRESSURE                                     │
│     Do mock interviews and timed practice                       │
│                                                                 │
│  SUCCESS = Pattern Recognition + Clean Code + Communication     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Complete DSA Guide Index

| Part | Topic | Key Concepts |
|------|-------|--------------|
| 1 | Fundamentals | Big O, Time/Space Complexity |
| 2 | Arrays & Strings | Two Pointers, Sliding Window, Binary Search |
| 3 | Hash Maps & Sets | Frequency Maps, Two Sum Pattern |
| 4 | Linked Lists, Stacks, Queues | Reversal, Fast/Slow, Monotonic Stack |
| 5 | Trees | Traversals, BST, LCA, Path Problems |
| 6 | Graphs | BFS, DFS, Topological Sort, Union-Find |
| 7 | Dynamic Programming | 1D, 2D, Knapsack, LCS |
| 8 | Advanced | Heaps, Tries, Segment Trees |
| 9 | Patterns & Strategy | Pattern Recognition, Interview Tips |

---

**Good luck with your interviews! Practice makes perfect.**
