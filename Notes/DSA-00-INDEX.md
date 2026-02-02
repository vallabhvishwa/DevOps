# Complete DSA Guide - Master Index
## Crack Any Coding Interview: Beginner to Advanced

---

## Guide Overview

This comprehensive 9-part guide covers everything you need to crack coding interviews at any company, from startups to FAANG.

---

## Part Index

### Beginner Level

| Part | Document | Topics | Key Patterns |
|------|----------|--------|--------------|
| 1 | [Fundamentals](./DSA-Complete-Guide-Part1-Fundamentals.md) | Big O, Time/Space Complexity | Complexity analysis rules |
| 2 | [Arrays & Strings](./DSA-Complete-Guide-Part2-Arrays-Strings.md) | Arrays, Strings, Searching | Two Pointers, Sliding Window, Binary Search, Prefix Sum |
| 3 | [Hash Maps & Sets](./DSA-Complete-Guide-Part3-HashMaps-Sets.md) | Dictionaries, Sets, Counter | Two Sum, Frequency Map, Group By |

### Intermediate Level

| Part | Document | Topics | Key Patterns |
|------|----------|--------|--------------|
| 4 | [Linked Lists, Stacks, Queues](./DSA-Complete-Guide-Part4-LinkedLists-Stacks-Queues.md) | Linked structures | Fast/Slow Pointers, Monotonic Stack, BFS Template |
| 5 | [Trees](./DSA-Complete-Guide-Part5-Trees.md) | Binary Trees, BST | Traversals, DFS, BST Operations, LCA |
| 6 | [Graphs](./DSA-Complete-Guide-Part6-Graphs.md) | Graph algorithms | BFS, DFS, Topological Sort, Union-Find |

### Advanced Level

| Part | Document | Topics | Key Patterns |
|------|----------|--------|--------------|
| 7 | [Dynamic Programming](./DSA-Complete-Guide-Part7-DynamicProgramming.md) | 1D DP, 2D DP | Memoization, Tabulation, Knapsack, LCS |
| 8 | [Advanced Data Structures](./DSA-Complete-Guide-Part8-Advanced.md) | Heaps, Tries, Union-Find | Priority Queue, Prefix Tree, Disjoint Set |
| 9 | [Problem-Solving Patterns](./DSA-Complete-Guide-Part9-Patterns.md) | Backtracking, Bit Manipulation, Strategy | Interview framework, 75 must-do problems |

---

## Quick Pattern Reference

```
PROBLEM TYPE                    → PATTERN/TECHNIQUE
─────────────────────────────────────────────────────
Sorted array search             → Binary Search
Find pair summing to target     → Two Pointers / Hash Map
Subarray with property          → Sliding Window
Contiguous maximum/minimum      → Kadane's / DP
Check for duplicates            → Hash Set
Count frequency                 → Hash Map / Counter

Level-by-level traversal        → BFS (Queue)
Shortest path (unweighted)      → BFS
All paths / combinations        → DFS / Backtracking
Connected components            → DFS / Union-Find
Cycle detection                 → DFS with colors
Dependency ordering             → Topological Sort

K largest/smallest              → Heap
Merge K sorted                  → Heap
Next greater/smaller            → Monotonic Stack
Prefix matching                 → Trie
Range queries with updates      → Segment Tree

Optimal substructure            → Dynamic Programming
Count number of ways            → Dynamic Programming
Minimum/maximum cost            → Dynamic Programming
All subsets                     → Backtracking
All permutations                → Backtracking
```

---

## 8-Week Study Plan

| Week | Focus | Parts | Problems |
|------|-------|-------|----------|
| 1-2 | Foundations | Parts 1-3 | 25 problems |
| 3 | Hash Tables | Part 3 | 15 problems |
| 4 | Linked Lists & Stacks | Part 4 | 15 problems |
| 5 | Trees | Part 5 | 15 problems |
| 6 | Graphs | Part 6 | 15 problems |
| 7 | Dynamic Programming | Part 7 | 20 problems |
| 8 | Advanced + Review | Parts 8-9 | 15 problems + mocks |

---

## Essential Commands Cheat Sheet

```python
# ARRAYS
arr.append(x)           # O(1)
arr.pop()               # O(1)
arr.sort()              # O(n log n)
sorted(arr, key=...)    # O(n log n)

# HASH MAP
d[key] = value          # O(1)
d.get(key, default)     # O(1)
key in d                # O(1)
Counter(arr)            # O(n)

# HEAP
heapq.heappush(h, x)    # O(log n)
heapq.heappop(h)        # O(log n)
heapq.heapify(arr)      # O(n)

# BINARY SEARCH
bisect.bisect_left(arr, x)
bisect.bisect_right(arr, x)

# DEQUE
from collections import deque
dq.append(x)            # O(1)
dq.appendleft(x)        # O(1)
dq.pop()                # O(1)
dq.popleft()            # O(1)
```

---

## Complexity Quick Reference

| Data Structure | Access | Search | Insert | Delete |
|----------------|--------|--------|--------|--------|
| Array | O(1) | O(n) | O(n) | O(n) |
| Sorted Array | O(1) | O(log n) | O(n) | O(n) |
| Hash Map/Set | - | O(1) | O(1) | O(1) |
| Heap | - | O(n) | O(log n) | O(log n) |
| BST (balanced) | - | O(log n) | O(log n) | O(log n) |
| Trie | - | O(m) | O(m) | O(m) |

| Algorithm | Time | Space |
|-----------|------|-------|
| Binary Search | O(log n) | O(1) |
| Merge Sort | O(n log n) | O(n) |
| Quick Sort | O(n log n) avg | O(log n) |
| BFS/DFS | O(V + E) | O(V) |
| Dijkstra | O((V+E) log V) | O(V) |
| Topological Sort | O(V + E) | O(V) |

---

## 75 Must-Do Problems

### Arrays & Strings (15)
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

### Linked Lists & Stacks (10)
16. Reverse Linked List
17. Merge Two Sorted Lists
18. Linked List Cycle
19. Remove Nth Node From End
20. Reorder List
21. Valid Parentheses
22. Min Stack
23. Daily Temperatures
24. Evaluate RPN
25. Sliding Window Maximum

### Trees (15)
26. Maximum Depth
27. Same Tree
28. Invert Binary Tree
29. Level Order Traversal
30. Validate BST
31. Kth Smallest in BST
32. LCA
33. Right Side View
34. Construct from Preorder/Inorder
35. Serialize/Deserialize
36. Diameter
37. Max Path Sum
38. Subtree of Another Tree
39. Path Sum
40. Implement Trie

### Graphs (10)
41. Number of Islands
42. Clone Graph
43. Course Schedule
44. Course Schedule II
45. Pacific Atlantic
46. Word Ladder
47. Connected Components
48. Graph Valid Tree
49. Alien Dictionary
50. Accounts Merge

### Heaps & Advanced (10)
51. Kth Largest Element
52. Top K Frequent
53. Find Median Data Stream
54. Merge K Sorted Lists
55. Task Scheduler
56. Subsets
57. Permutations
58. Combination Sum
59. Letter Combinations
60. Word Search

### Dynamic Programming (15)
61. Climbing Stairs
62. House Robber
63. House Robber II
64. Coin Change
65. LIS
66. Word Break
67. Unique Paths
68. LCS
69. Edit Distance
70. Jump Game
71. Partition Equal Subset
72. Decode Ways
73. Max Product Subarray
74. Target Sum
75. Palindromic Substrings

---

## Statistics

| Metric | Value |
|--------|-------|
| Total Parts | 9 |
| Total Lines | ~12,000+ |
| Patterns Covered | 15+ |
| Problems Referenced | 150+ |
| Must-Do Problems | 75 |

---

## Interview Day Checklist

```
BEFORE:
□ Review pattern cheat sheet
□ Review complexity table
□ Practice 1-2 warm-up problems
□ Get good sleep

DURING:
□ Ask clarifying questions
□ Discuss approach before coding
□ Talk through your thought process
□ Handle edge cases
□ Test your code
□ Discuss complexity

AFTER:
□ Thank the interviewer
□ Review what went well/poorly
□ Note areas to improve
```

---

**Total Preparation Time: 8-12 weeks of consistent practice**

**Good luck! You've got this! 🎯**
