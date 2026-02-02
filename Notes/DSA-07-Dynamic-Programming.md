# Complete DSA Guide - Crack Any Coding Interview
# Part 7: Dynamic Programming

---

## Table of Contents

1. [DP Fundamentals](#1-dp-fundamentals)
2. [1D Dynamic Programming](#2-1d-dynamic-programming)
3. [2D Dynamic Programming](#3-2d-dynamic-programming)
4. [Common DP Patterns](#4-common-dp-patterns)
5. [Advanced DP](#5-advanced-dp)
6. [Practice Problems](#6-practice-problems)

---

## 1. DP Fundamentals

### 1.1 What is Dynamic Programming?

```
Dynamic Programming:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  DP = Recursion + Memoization (or Bottom-Up Tabulation)         │
│                                                                 │
│  TWO KEY PROPERTIES:                                            │
│                                                                 │
│  1. OPTIMAL SUBSTRUCTURE                                        │
│     Solution to problem can be built from solutions             │
│     to smaller subproblems.                                     │
│                                                                 │
│  2. OVERLAPPING SUBPROBLEMS                                     │
│     Same subproblems are solved multiple times.                 │
│                                                                 │
│  Example - Fibonacci:                                           │
│                                                                 │
│  fib(5)                                                         │
│  ├── fib(4)                                                     │
│  │   ├── fib(3) ← Computed twice!                               │
│  │   │   ├── fib(2)                                             │
│  │   │   └── fib(1)                                             │
│  │   └── fib(2) ← Computed twice!                               │
│  └── fib(3) ← Same subproblem                                   │
│      ├── fib(2)                                                 │
│      └── fib(1)                                                 │
│                                                                 │
│  Without DP: O(2ⁿ) - exponential                                │
│  With DP: O(n) - linear                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Two Approaches

```python
# APPROACH 1: Top-Down (Memoization)
# Start from main problem, recurse to subproblems, cache results

def fibonacci_memoization(n, memo={}):
    """Top-down with memoization."""
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    
    memo[n] = fibonacci_memoization(n - 1, memo) + fibonacci_memoization(n - 2, memo)
    return memo[n]


# APPROACH 2: Bottom-Up (Tabulation)
# Start from smallest subproblems, build up to main problem

def fibonacci_tabulation(n):
    """Bottom-up with tabulation."""
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    
    return dp[n]


# APPROACH 3: Space-Optimized Bottom-Up
# When dp[i] only depends on previous few values

def fibonacci_optimized(n):
    """O(1) space."""
    if n <= 1:
        return n
    
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr
    
    return curr
```

### 1.3 How to Approach DP Problems

```
DP Problem-Solving Framework:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. DEFINE STATE                                                │
│     What variables define a unique subproblem?                  │
│     dp[i] = "answer for first i elements"                       │
│     dp[i][j] = "answer for substring from i to j"               │
│                                                                 │
│  2. FIND RECURRENCE (Transition)                                │
│     How does dp[i] relate to smaller subproblems?               │
│     dp[i] = max(dp[i-1], dp[i-2] + arr[i])                      │
│                                                                 │
│  3. IDENTIFY BASE CASES                                         │
│     What are the simplest subproblems we know the answer to?    │
│     dp[0] = 0, dp[1] = arr[0]                                   │
│                                                                 │
│  4. DETERMINE COMPUTATION ORDER                                 │
│     In what order to fill the dp table?                         │
│     Usually: left to right, or smaller to larger                │
│                                                                 │
│  5. OPTIMIZE SPACE (if possible)                                │
│     Can we reduce from O(n) to O(1)?                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 1D Dynamic Programming

### 2.1 Climbing Stairs

```python
"""
Problem: Ways to climb n stairs (1 or 2 steps at a time).
Input: n = 4
Output: 5 (1+1+1+1, 1+1+2, 1+2+1, 2+1+1, 2+2)
"""

def climb_stairs(n):
    """
    dp[i] = ways to reach step i
    dp[i] = dp[i-1] + dp[i-2] (from step i-1 or i-2)
    
    Time: O(n), Space: O(1)
    """
    if n <= 2:
        return n
    
    prev, curr = 1, 2
    
    for _ in range(3, n + 1):
        prev, curr = curr, prev + curr
    
    return curr
```

### 2.2 House Robber

```python
"""
Problem: Max money from robbing houses (can't rob adjacent).
Input: nums = [2, 7, 9, 3, 1]
Output: 12 (rob houses 0, 2, 4: 2+9+1)
"""

def rob(nums):
    """
    dp[i] = max money from first i houses
    dp[i] = max(dp[i-1], dp[i-2] + nums[i])
    
    Time: O(n), Space: O(1)
    """
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]
    
    prev2, prev1 = 0, 0
    
    for num in nums:
        curr = max(prev1, prev2 + num)
        prev2, prev1 = prev1, curr
    
    return prev1


# House Robber II (circular - first and last are adjacent)
def rob_circular(nums):
    """Rob either houses[0:n-1] or houses[1:n]."""
    if len(nums) == 1:
        return nums[0]
    
    return max(rob(nums[:-1]), rob(nums[1:]))
```

### 2.3 Coin Change

```python
"""
Problem: Minimum coins to make amount.
Input: coins = [1, 2, 5], amount = 11
Output: 3 (5 + 5 + 1)
"""

def coin_change(coins, amount):
    """
    dp[i] = min coins to make amount i
    dp[i] = min(dp[i], dp[i - coin] + 1) for each coin
    
    Time: O(amount * len(coins)), Space: O(amount)
    """
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i and dp[i - coin] != float('inf'):
                dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1


# Variation: Count ways to make amount
def coin_change_ways(coins, amount):
    """
    dp[i] = number of ways to make amount i
    """
    dp = [0] * (amount + 1)
    dp[0] = 1
    
    for coin in coins:  # Order matters for counting combinations
        for i in range(coin, amount + 1):
            dp[i] += dp[i - coin]
    
    return dp[amount]
```

### 2.4 Longest Increasing Subsequence

```python
"""
Problem: Length of longest strictly increasing subsequence.
Input: nums = [10, 9, 2, 5, 3, 7, 101, 18]
Output: 4 ([2, 3, 7, 101] or [2, 5, 7, 101])
"""

def length_of_lis(nums):
    """
    dp[i] = length of LIS ending at index i
    
    Time: O(n²), Space: O(n)
    """
    if not nums:
        return 0
    
    n = len(nums)
    dp = [1] * n
    
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)


# Optimized with binary search: O(n log n)
import bisect

def length_of_lis_optimized(nums):
    """
    Maintain array of smallest tail for each length.
    """
    tails = []
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)
```

### 2.5 Word Break

```python
"""
Problem: Can string be segmented into dictionary words?
Input: s = "leetcode", wordDict = ["leet", "code"]
Output: True
"""

def word_break(s, wordDict):
    """
    dp[i] = True if s[:i] can be segmented
    
    Time: O(n² * m), Space: O(n)
    """
    word_set = set(wordDict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[n]
```

---

## 3. 2D Dynamic Programming

### 3.1 Unique Paths

```python
"""
Problem: Paths from top-left to bottom-right (only right/down).
Input: m = 3, n = 7
Output: 28
"""

def unique_paths(m, n):
    """
    dp[i][j] = paths to reach cell (i, j)
    dp[i][j] = dp[i-1][j] + dp[i][j-1]
    
    Time: O(m*n), Space: O(n)
    """
    dp = [1] * n
    
    for i in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j - 1]
    
    return dp[-1]


# Unique Paths II (with obstacles)
def unique_paths_with_obstacles(grid):
    if grid[0][0] == 1:
        return 0
    
    m, n = len(grid), len(grid[0])
    dp = [0] * n
    dp[0] = 1
    
    for i in range(m):
        for j in range(n):
            if grid[i][j] == 1:
                dp[j] = 0
            elif j > 0:
                dp[j] += dp[j - 1]
    
    return dp[-1]
```

### 3.2 Minimum Path Sum

```python
"""
Problem: Min path sum from top-left to bottom-right.
Input: grid = [[1,3,1],[1,5,1],[4,2,1]]
Output: 7 (1→3→1→1→1)
"""

def min_path_sum(grid):
    """
    dp[i][j] = min sum to reach (i, j)
    
    Time: O(m*n), Space: O(1) if modify in-place
    """
    m, n = len(grid), len(grid[0])
    
    # Modify grid in-place
    for i in range(m):
        for j in range(n):
            if i == 0 and j == 0:
                continue
            elif i == 0:
                grid[i][j] += grid[i][j - 1]
            elif j == 0:
                grid[i][j] += grid[i - 1][j]
            else:
                grid[i][j] += min(grid[i - 1][j], grid[i][j - 1])
    
    return grid[-1][-1]
```

### 3.3 Longest Common Subsequence

```python
"""
Problem: Length of longest common subsequence.
Input: text1 = "abcde", text2 = "ace"
Output: 3 ("ace")
"""

def longest_common_subsequence(text1, text2):
    """
    dp[i][j] = LCS of text1[:i] and text2[:j]
    
    Time: O(m*n), Space: O(n)
    """
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    return dp[m][n]


# Space optimized
def lcs_optimized(text1, text2):
    if len(text1) < len(text2):
        text1, text2 = text2, text1
    
    prev = [0] * (len(text2) + 1)
    
    for i in range(1, len(text1) + 1):
        curr = [0] * (len(text2) + 1)
        for j in range(1, len(text2) + 1):
            if text1[i - 1] == text2[j - 1]:
                curr[j] = prev[j - 1] + 1
            else:
                curr[j] = max(prev[j], curr[j - 1])
        prev = curr
    
    return prev[-1]
```

### 3.4 Edit Distance

```python
"""
Problem: Min operations to convert word1 to word2.
Operations: insert, delete, replace
Input: word1 = "horse", word2 = "ros"
Output: 3 (horse → rorse → rose → ros)
"""

def min_distance(word1, word2):
    """
    dp[i][j] = min edits for word1[:i] → word2[:j]
    
    Time: O(m*n), Space: O(n)
    """
    m, n = len(word1), len(word2)
    
    # Base case: converting empty string
    prev = list(range(n + 1))
    
    for i in range(1, m + 1):
        curr = [i] + [0] * n
        
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                curr[j] = prev[j - 1]  # No edit needed
            else:
                curr[j] = 1 + min(
                    prev[j],      # Delete
                    curr[j - 1],  # Insert
                    prev[j - 1]   # Replace
                )
        
        prev = curr
    
    return prev[-1]
```

### 3.5 0/1 Knapsack

```python
"""
Problem: Max value in knapsack with capacity W.
Each item can be taken at most once.
"""

def knapsack_01(weights, values, W):
    """
    dp[i][w] = max value using first i items with capacity w
    
    Time: O(n*W), Space: O(W)
    """
    n = len(weights)
    dp = [0] * (W + 1)
    
    for i in range(n):
        # Iterate backwards to avoid using item multiple times
        for w in range(W, weights[i] - 1, -1):
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[W]


# Unbounded Knapsack (items can be used unlimited times)
def knapsack_unbounded(weights, values, W):
    dp = [0] * (W + 1)
    
    for w in range(1, W + 1):
        for i in range(len(weights)):
            if weights[i] <= w:
                dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[W]
```

---

## 4. Common DP Patterns

### 4.1 Pattern Recognition

```
DP Pattern Categories:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. LINEAR SEQUENCE                                             │
│     dp[i] depends on dp[i-1], dp[i-2], etc.                     │
│     Examples: Climbing Stairs, House Robber, Decode Ways        │
│                                                                 │
│  2. TWO SEQUENCES                                               │
│     dp[i][j] for position i in seq1, j in seq2                  │
│     Examples: LCS, Edit Distance                                │
│                                                                 │
│  3. GRID/MATRIX                                                 │
│     dp[i][j] for cell (i, j) in grid                            │
│     Examples: Unique Paths, Min Path Sum                        │
│                                                                 │
│  4. INTERVAL/RANGE                                              │
│     dp[i][j] for substring/subarray from i to j                 │
│     Examples: Longest Palindromic Substring, Matrix Chain       │
│                                                                 │
│  5. SUBSET/KNAPSACK                                             │
│     dp[i][w] for first i items with capacity/sum w              │
│     Examples: 0/1 Knapsack, Partition Equal Subset Sum          │
│                                                                 │
│  6. DECISION MAKING                                             │
│     Multiple states tracking different decisions                │
│     Examples: Best Time to Buy/Sell Stock with Cooldown         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Partition Equal Subset Sum

```python
"""
Problem: Can array be partitioned into two equal sum subsets?
Input: nums = [1, 5, 11, 5]
Output: True ([1, 5, 5] and [11])
"""

def can_partition(nums):
    """
    Reduce to: can we find subset summing to total/2?
    
    Time: O(n * sum), Space: O(sum)
    """
    total = sum(nums)
    
    if total % 2 != 0:
        return False
    
    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True
    
    for num in nums:
        for j in range(target, num - 1, -1):
            dp[j] = dp[j] or dp[j - num]
    
    return dp[target]
```

### 4.3 Longest Palindromic Substring (DP)

```python
"""
Problem: Find longest palindromic substring.
"""

def longest_palindrome(s):
    """
    dp[i][j] = True if s[i:j+1] is palindrome
    
    Time: O(n²), Space: O(n²)
    """
    n = len(s)
    if n < 2:
        return s
    
    dp = [[False] * n for _ in range(n)]
    start, max_len = 0, 1
    
    # All single characters are palindromes
    for i in range(n):
        dp[i][i] = True
    
    # Check substrings of length 2+
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            
            if length == 2:
                dp[i][j] = (s[i] == s[j])
            else:
                dp[i][j] = (s[i] == s[j]) and dp[i + 1][j - 1]
            
            if dp[i][j] and length > max_len:
                start = i
                max_len = length
    
    return s[start:start + max_len]
```

---

## 5. Advanced DP

### 5.1 Best Time to Buy/Sell Stock III

```python
"""
Problem: Max profit with at most 2 transactions.
"""

def max_profit(prices):
    """
    Track 4 states: after 1st buy, 1st sell, 2nd buy, 2nd sell
    
    Time: O(n), Space: O(1)
    """
    buy1 = buy2 = float('-inf')
    sell1 = sell2 = 0
    
    for price in prices:
        buy1 = max(buy1, -price)
        sell1 = max(sell1, buy1 + price)
        buy2 = max(buy2, sell1 - price)
        sell2 = max(sell2, buy2 + price)
    
    return sell2
```

### 5.2 Target Sum

```python
"""
Problem: Ways to assign +/- to nums to reach target.
Input: nums = [1, 1, 1, 1, 1], target = 3
Output: 5
"""

def find_target_sum_ways(nums, target):
    """
    Reduce to subset sum: find subset P where P - (S-P) = target
    So P = (S + target) / 2
    
    Time: O(n * sum), Space: O(sum)
    """
    total = sum(nums)
    
    if (total + target) % 2 != 0 or abs(target) > total:
        return 0
    
    subset_sum = (total + target) // 2
    dp = [0] * (subset_sum + 1)
    dp[0] = 1
    
    for num in nums:
        for j in range(subset_sum, num - 1, -1):
            dp[j] += dp[j - num]
    
    return dp[subset_sum]
```

---

## 6. Practice Problems

```
EASY:
├── 70. Climbing Stairs ⭐
├── 746. Min Cost Climbing Stairs
├── 121. Best Time to Buy/Sell Stock

MEDIUM:
├── 198. House Robber ⭐
├── 213. House Robber II
├── 322. Coin Change ⭐
├── 300. Longest Increasing Subsequence ⭐
├── 139. Word Break
├── 62. Unique Paths
├── 64. Minimum Path Sum
├── 1143. Longest Common Subsequence ⭐
├── 72. Edit Distance ⭐
├── 416. Partition Equal Subset Sum
├── 494. Target Sum
├── 5. Longest Palindromic Substring
├── 647. Palindromic Substrings

HARD:
├── 123. Best Time to Buy/Sell Stock III
├── 188. Best Time to Buy/Sell Stock IV
├── 309. Best Time to Buy/Sell with Cooldown
├── 10. Regular Expression Matching
├── 44. Wildcard Matching
├── 312. Burst Balloons
├── 115. Distinct Subsequences
```

---

## Summary

1. **Identify DP**: Optimal substructure + overlapping subproblems
2. **Define state**: What variables uniquely identify a subproblem?
3. **Find transition**: How does current state relate to previous?
4. **Set base cases**: Simplest subproblems with known answers
5. **Optimize space**: Often can reduce from 2D to 1D

Key patterns:
- Linear DP, Two sequences, Grid, Interval, Knapsack

---

**Next Part**: [Part 8: Advanced Data Structures](./DSA-Complete-Guide-Part8-Advanced.md)
