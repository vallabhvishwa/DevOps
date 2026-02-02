# Complete DSA Guide - Crack Any Coding Interview
# Part 2: Arrays & Strings

---

## Table of Contents

1. [Array Fundamentals](#1-array-fundamentals)
2. [Two Pointers Pattern](#2-two-pointers-pattern)
3. [Sliding Window Pattern](#3-sliding-window-pattern)
4. [Binary Search Pattern](#4-binary-search-pattern)
5. [Prefix Sum Pattern](#5-prefix-sum-pattern)
6. [String Techniques](#6-string-techniques)
7. [Practice Problems](#7-practice-problems)

---

## 1. Array Fundamentals

### 1.1 Array Basics in Python

```python
# Creating arrays (lists in Python)
arr = [1, 2, 3, 4, 5]
arr = [0] * 10              # [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
arr = list(range(5))        # [0, 1, 2, 3, 4]
arr = [[0] * 3 for _ in range(3)]  # 3x3 matrix

# Accessing elements
first = arr[0]              # First element
last = arr[-1]              # Last element
second_last = arr[-2]       # Second to last

# Slicing
arr[1:4]                    # Elements from index 1 to 3 (exclusive)
arr[:3]                     # First 3 elements
arr[3:]                     # From index 3 to end
arr[::2]                    # Every 2nd element
arr[::-1]                   # Reverse array

# Common operations
len(arr)                    # Length
arr.append(6)               # Add to end - O(1)
arr.pop()                   # Remove from end - O(1)
arr.insert(0, 0)            # Insert at index - O(n)
arr.remove(3)               # Remove by value - O(n)
arr.reverse()               # Reverse in-place
arr.sort()                  # Sort in-place
sorted_arr = sorted(arr)    # Return new sorted array

# Searching
3 in arr                    # Check existence - O(n)
arr.index(3)                # Find index - O(n)
arr.count(3)                # Count occurrences - O(n)

# Useful functions
sum(arr)                    # Sum of elements
min(arr)                    # Minimum element
max(arr)                    # Maximum element
```

### 1.2 2D Arrays (Matrices)

```python
# Creating 2D arrays
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

rows = len(matrix)          # Number of rows
cols = len(matrix[0])       # Number of columns

# Accessing elements
element = matrix[row][col]

# Iterating
for i in range(rows):
    for j in range(cols):
        print(matrix[i][j])

# Common directions (for traversal problems)
directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]  # Right, Left, Down, Up
all_directions = [(0, 1), (0, -1), (1, 0), (-1, 0),
                  (1, 1), (1, -1), (-1, 1), (-1, -1)]  # 8 directions

# Check bounds
def is_valid(row, col, rows, cols):
    return 0 <= row < rows and 0 <= col < cols
```

---

## 2. Two Pointers Pattern

### 2.1 Concept

```
Two Pointers Pattern:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Use two indices to traverse the array efficiently.             │
│                                                                 │
│  TYPES:                                                         │
│                                                                 │
│  1. OPPOSITE DIRECTION (converging):                            │
│     [1, 2, 3, 4, 5, 6, 7]                                       │
│      ↑                 ↑                                        │
│     left             right                                      │
│     Move towards each other                                     │
│                                                                 │
│  2. SAME DIRECTION:                                             │
│     [1, 2, 3, 4, 5, 6, 7]                                       │
│      ↑  ↑                                                       │
│     slow fast                                                   │
│     One moves faster than the other                             │
│                                                                 │
│  WHEN TO USE:                                                   │
│  ├── Sorted array problems                                      │
│  ├── Finding pairs with sum/difference                          │
│  ├── Palindrome checking                                        │
│  ├── Removing duplicates in-place                               │
│  └── Partitioning arrays                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Two Sum II (Sorted Array)

```python
"""
Problem: Given a sorted array, find two numbers that sum to target.
Input: numbers = [2, 7, 11, 15], target = 9
Output: [1, 2] (indices are 1-indexed)
"""

def two_sum_sorted(numbers, target):
    """
    Use two pointers from both ends.
    If sum too small → move left pointer right
    If sum too big → move right pointer left
    
    Time: O(n), Space: O(1)
    """
    left, right = 0, len(numbers) - 1
    
    while left < right:
        current_sum = numbers[left] + numbers[right]
        
        if current_sum == target:
            return [left + 1, right + 1]  # 1-indexed
        elif current_sum < target:
            left += 1   # Need bigger sum
        else:
            right -= 1  # Need smaller sum
    
    return []  # No solution found


# Example walkthrough:
# numbers = [2, 7, 11, 15], target = 9
# left=0, right=3: 2+15=17 > 9, right--
# left=0, right=2: 2+11=13 > 9, right--
# left=0, right=1: 2+7=9 == 9, return [1, 2]
```

### 2.3 Three Sum

```python
"""
Problem: Find all unique triplets that sum to zero.
Input: nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
"""

def three_sum(nums):
    """
    Sort array, then for each number, use two pointers
    to find pairs that sum to negative of that number.
    
    Time: O(n²), Space: O(1) excluding output
    """
    nums.sort()
    result = []
    n = len(nums)
    
    for i in range(n - 2):
        # Skip duplicates for first number
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        # Two pointer search
        left, right = i + 1, n - 1
        target = -nums[i]
        
        while left < right:
            current_sum = nums[left] + nums[right]
            
            if current_sum == target:
                result.append([nums[i], nums[left], nums[right]])
                
                # Skip duplicates
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                
                left += 1
                right -= 1
            elif current_sum < target:
                left += 1
            else:
                right -= 1
    
    return result
```

### 2.4 Container With Most Water

```python
"""
Problem: Find two lines that form container with most water.
Input: height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
Output: 49 (between lines at index 1 and 8)
"""

def max_area(height):
    """
    Start with widest container (left and right edges).
    Move the shorter line inward (might find taller line).
    
    Time: O(n), Space: O(1)
    """
    left, right = 0, len(height) - 1
    max_water = 0
    
    while left < right:
        # Calculate current area
        width = right - left
        h = min(height[left], height[right])
        area = width * h
        max_water = max(max_water, area)
        
        # Move shorter line (taller might increase area)
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water
```

### 2.5 Valid Palindrome

```python
"""
Problem: Check if string is palindrome (ignore non-alphanumeric).
Input: s = "A man, a plan, a canal: Panama"
Output: True
"""

def is_palindrome(s):
    """
    Use two pointers from both ends, skip non-alphanumeric.
    
    Time: O(n), Space: O(1)
    """
    left, right = 0, len(s) - 1
    
    while left < right:
        # Skip non-alphanumeric
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        
        # Compare (case-insensitive)
        if s[left].lower() != s[right].lower():
            return False
        
        left += 1
        right -= 1
    
    return True
```

### 2.6 Remove Duplicates (In-Place)

```python
"""
Problem: Remove duplicates from sorted array in-place.
Input: nums = [1, 1, 2, 2, 3]
Output: 3 (first 3 elements are [1, 2, 3])
"""

def remove_duplicates(nums):
    """
    Use slow pointer for position to write,
    fast pointer to scan through array.
    
    Time: O(n), Space: O(1)
    """
    if not nums:
        return 0
    
    slow = 0  # Position of last unique element
    
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:
            slow += 1
            nums[slow] = nums[fast]
    
    return slow + 1  # Length of unique portion


# Variation: Remove all instances of a value
def remove_element(nums, val):
    """Remove all instances of val, return new length."""
    slow = 0
    
    for fast in range(len(nums)):
        if nums[fast] != val:
            nums[slow] = nums[fast]
            slow += 1
    
    return slow
```

---

## 3. Sliding Window Pattern

### 3.1 Concept

```
Sliding Window Pattern:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Maintain a "window" over a portion of the array.               │
│  Slide the window to examine different subarrays.               │
│                                                                 │
│  FIXED SIZE WINDOW:                                             │
│  [1, 2, 3, 4, 5, 6, 7]                                          │
│   └──────┘              Window size = 3                         │
│      └──────┘           Slide right                             │
│         └──────┘                                                │
│                                                                 │
│  VARIABLE SIZE WINDOW:                                          │
│  [1, 2, 3, 4, 5, 6, 7]                                          │
│   └──┘                  Expand right                            │
│   └─────┘               Expand more                             │
│      └──────┘           Shrink from left                        │
│                                                                 │
│  WHEN TO USE:                                                   │
│  ├── Contiguous subarray/substring problems                     │
│  ├── Maximum/minimum in subarray of size k                      │
│  ├── Longest substring with condition                           │
│  └── Subarray sum equals target                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Fixed Size Window Template

```python
def fixed_window_template(arr, k):
    """
    Template for fixed size window problems.
    
    Time: O(n), Space: O(1) or O(k)
    """
    n = len(arr)
    if n < k:
        return None
    
    # Initialize window with first k elements
    window_sum = sum(arr[:k])  # Or whatever state you need
    result = window_sum
    
    # Slide window
    for i in range(k, n):
        # Add new element (entering window)
        window_sum += arr[i]
        # Remove old element (leaving window)
        window_sum -= arr[i - k]
        # Update result
        result = max(result, window_sum)
    
    return result
```

### 3.3 Maximum Sum Subarray of Size K

```python
"""
Problem: Find maximum sum of any contiguous subarray of size k.
Input: arr = [2, 1, 5, 1, 3, 2], k = 3
Output: 9 (subarray [5, 1, 3])
"""

def max_sum_subarray(arr, k):
    """
    Fixed size sliding window.
    
    Time: O(n), Space: O(1)
    """
    n = len(arr)
    if n < k:
        return 0
    
    # Calculate sum of first window
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    # Slide window: add right, remove left
    for i in range(k, n):
        window_sum += arr[i] - arr[i - k]
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```

### 3.4 Variable Size Window Template

```python
def variable_window_template(arr):
    """
    Template for variable size window problems.
    
    Time: O(n), Space: depends on problem
    """
    left = 0
    result = 0
    window_state = ...  # dict, set, sum, etc.
    
    for right in range(len(arr)):
        # Expand window: add arr[right] to window
        update_window_state(arr[right])
        
        # Shrink window while condition violated
        while window_is_invalid():
            # Remove arr[left] from window
            remove_from_window(arr[left])
            left += 1
        
        # Update result (window is now valid)
        result = update_result(result, right - left + 1)
    
    return result
```

### 3.5 Longest Substring Without Repeating Characters

```python
"""
Problem: Find length of longest substring without repeating characters.
Input: s = "abcabcbb"
Output: 3 ("abc")
"""

def length_of_longest_substring(s):
    """
    Variable window with set to track characters in window.
    
    Time: O(n), Space: O(min(n, alphabet_size))
    """
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        # Shrink window until no duplicate
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        
        # Add current character
        char_set.add(s[right])
        
        # Update result
        max_length = max(max_length, right - left + 1)
    
    return max_length


# Alternative using dict for last seen index
def length_of_longest_substring_v2(s):
    """Using hash map to jump left pointer directly."""
    last_seen = {}
    left = 0
    max_length = 0
    
    for right, char in enumerate(s):
        if char in last_seen and last_seen[char] >= left:
            left = last_seen[char] + 1
        
        last_seen[char] = right
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

### 3.6 Minimum Window Substring

```python
"""
Problem: Find minimum window in s containing all characters of t.
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
"""

from collections import Counter

def min_window(s, t):
    """
    Variable window: expand until all chars found,
    then shrink to find minimum.
    
    Time: O(n), Space: O(m) where m = len(t)
    """
    if not s or not t:
        return ""
    
    # Count required characters
    required = Counter(t)
    required_count = len(required)
    
    # Window tracking
    window_counts = {}
    formed = 0  # Number of unique chars with required frequency
    
    left = 0
    min_len = float('inf')
    min_left = 0
    
    for right in range(len(s)):
        # Add character to window
        char = s[right]
        window_counts[char] = window_counts.get(char, 0) + 1
        
        # Check if this character's count matches required
        if char in required and window_counts[char] == required[char]:
            formed += 1
        
        # Shrink window while valid
        while formed == required_count:
            # Update result if smaller
            if right - left + 1 < min_len:
                min_len = right - left + 1
                min_left = left
            
            # Remove left character
            left_char = s[left]
            window_counts[left_char] -= 1
            if left_char in required and window_counts[left_char] < required[left_char]:
                formed -= 1
            left += 1
    
    return "" if min_len == float('inf') else s[min_left:min_left + min_len]
```

### 3.7 Subarray Sum Equals K

```python
"""
Problem: Find number of contiguous subarrays that sum to k.
Input: nums = [1, 2, 3], k = 3
Output: 2 ([1, 2] and [3])

Note: This uses prefix sum + hash map, not pure sliding window.
      Sliding window only works for positive numbers.
"""

def subarray_sum(nums, k):
    """
    For each position, check how many prefix sums differ by k.
    
    If prefix_sum[j] - prefix_sum[i] = k,
    then subarray from i+1 to j sums to k.
    
    Time: O(n), Space: O(n)
    """
    count = 0
    current_sum = 0
    prefix_sums = {0: 1}  # prefix sum -> frequency
    
    for num in nums:
        current_sum += num
        
        # Check if (current_sum - k) was seen before
        if current_sum - k in prefix_sums:
            count += prefix_sums[current_sum - k]
        
        # Record current prefix sum
        prefix_sums[current_sum] = prefix_sums.get(current_sum, 0) + 1
    
    return count
```

---

## 4. Binary Search Pattern

### 4.1 Concept

```
Binary Search:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Repeatedly divide search space in half.                        │
│  REQUIRES: Sorted data or monotonic property.                   │
│                                                                 │
│  [1, 3, 5, 7, 9, 11, 13, 15, 17]                                 │
│   ↑           ↑              ↑                                  │
│  left        mid           right                                │
│                                                                 │
│  Target = 9:                                                    │
│  mid = 9? Yes! Found.                                           │
│                                                                 │
│  Target = 3:                                                    │
│  mid = 9 > 3, search left half                                  │
│  [1, 3, 5, 7]                                                   │
│   ↑  ↑     ↑                                                    │
│  mid = 5 > 3, search left half                                  │
│  [1, 3]                                                         │
│  mid = 3? Yes! Found.                                           │
│                                                                 │
│  Time: O(log n) - halving each time                             │
│  Space: O(1) iterative, O(log n) recursive                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Standard Binary Search

```python
def binary_search(arr, target):
    """
    Find target in sorted array. Return index or -1.
    
    Time: O(log n), Space: O(1)
    """
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = left + (right - left) // 2  # Avoid overflow
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1
```

### 4.3 Binary Search Variations

```python
def find_first_occurrence(arr, target):
    """Find first (leftmost) occurrence of target."""
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            result = mid
            right = mid - 1  # Keep searching left
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return result


def find_last_occurrence(arr, target):
    """Find last (rightmost) occurrence of target."""
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            result = mid
            left = mid + 1  # Keep searching right
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return result


def find_insert_position(arr, target):
    """Find where target should be inserted (bisect_left)."""
    left, right = 0, len(arr)
    
    while left < right:
        mid = left + (right - left) // 2
        
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid
    
    return left
```

### 4.4 Search in Rotated Sorted Array

```python
"""
Problem: Search in rotated sorted array.
Input: nums = [4, 5, 6, 7, 0, 1, 2], target = 0
Output: 4
"""

def search_rotated(nums, target):
    """
    Modified binary search. One half is always sorted.
    
    Time: O(log n), Space: O(1)
    """
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        
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

### 4.5 Find Peak Element

```python
"""
Problem: Find a peak element (greater than neighbors).
Input: nums = [1, 2, 3, 1]
Output: 2 (index of peak 3)
"""

def find_peak_element(nums):
    """
    Binary search: go towards higher neighbor.
    
    Time: O(log n), Space: O(1)
    """
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] < nums[mid + 1]:
            left = mid + 1  # Peak is on right
        else:
            right = mid     # Peak is on left or at mid
    
    return left
```

### 4.6 Binary Search on Answer

```python
"""
Problem: Koko eating bananas - find minimum eating speed.
Input: piles = [3, 6, 7, 11], h = 8 hours
Output: 4 bananas/hour
"""

def min_eating_speed(piles, h):
    """
    Binary search on the answer (speed).
    
    Time: O(n log m) where m = max(piles)
    """
    import math
    
    def can_finish(speed):
        """Check if Koko can finish at this speed."""
        hours = sum(math.ceil(pile / speed) for pile in piles)
        return hours <= h
    
    left, right = 1, max(piles)
    
    while left < right:
        mid = left + (right - left) // 2
        
        if can_finish(mid):
            right = mid     # Try smaller speed
        else:
            left = mid + 1  # Need faster speed
    
    return left
```

---

## 5. Prefix Sum Pattern

### 5.1 Concept

```
Prefix Sum:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Precompute cumulative sums to answer range queries in O(1).    │
│                                                                 │
│  Array:      [1,  2,  3,  4,  5]                                │
│  Prefix Sum: [0,  1,  3,  6, 10, 15]                            │
│               ↑                                                 │
│               0 is added at start for easier calculations       │
│                                                                 │
│  Sum(i, j) = prefix[j+1] - prefix[i]                            │
│                                                                 │
│  Example: Sum from index 1 to 3 (elements 2, 3, 4)              │
│  = prefix[4] - prefix[1] = 10 - 1 = 9                           │
│                                                                 │
│  WHEN TO USE:                                                   │
│  ├── Range sum queries                                          │
│  ├── Subarray sum problems                                      │
│  └── 2D matrix sum queries                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Range Sum Query

```python
class NumArray:
    """
    Range sum query - immutable array.
    
    Build: O(n), Query: O(1)
    """
    
    def __init__(self, nums):
        self.prefix = [0]
        for num in nums:
            self.prefix.append(self.prefix[-1] + num)
    
    def sum_range(self, left, right):
        """Sum of elements from left to right (inclusive)."""
        return self.prefix[right + 1] - self.prefix[left]


# Usage:
# arr = NumArray([1, 2, 3, 4, 5])
# arr.sum_range(1, 3)  # Returns 9 (2 + 3 + 4)
```

### 5.3 Product of Array Except Self

```python
"""
Problem: Return array where output[i] is product of all except nums[i].
Input: nums = [1, 2, 3, 4]
Output: [24, 12, 8, 6]
Constraint: No division!
"""

def product_except_self(nums):
    """
    Use prefix and suffix products.
    
    Time: O(n), Space: O(1) excluding output
    """
    n = len(nums)
    result = [1] * n
    
    # First pass: prefix products
    prefix = 1
    for i in range(n):
        result[i] = prefix
        prefix *= nums[i]
    
    # Second pass: multiply by suffix products
    suffix = 1
    for i in range(n - 1, -1, -1):
        result[i] *= suffix
        suffix *= nums[i]
    
    return result

# Walkthrough for [1, 2, 3, 4]:
# After prefix pass: [1, 1, 2, 6]
# After suffix pass: [24, 12, 8, 6]
```

---

## 6. String Techniques

### 6.1 String Basics in Python

```python
# String operations
s = "hello world"

len(s)                      # Length
s[0]                        # First character
s[-1]                       # Last character
s[1:4]                      # Substring (slicing)

# Immutability - strings cannot be modified
# s[0] = 'H'  # ERROR!

# Convert to list for modification
chars = list(s)
chars[0] = 'H'
s = ''.join(chars)          # "Hello world"

# Common string methods
s.lower()                   # Lowercase
s.upper()                   # Uppercase
s.strip()                   # Remove whitespace from ends
s.split()                   # Split by whitespace
s.split(',')                # Split by comma
''.join(['a', 'b', 'c'])    # Join list to string
s.replace('l', 'x')         # Replace all occurrences
s.find('world')             # Find substring index (-1 if not found)
s.count('l')                # Count occurrences
s.startswith('hello')       # Check prefix
s.endswith('world')         # Check suffix
s.isalpha()                 # Check if all alphabetic
s.isdigit()                 # Check if all digits
s.isalnum()                 # Check if all alphanumeric

# Character operations
ord('a')                    # 97 (ASCII value)
chr(97)                     # 'a' (character from ASCII)
```

### 6.2 Common String Patterns

```python
# Reverse a string
s[::-1]

# Check palindrome
s == s[::-1]

# Count character frequency
from collections import Counter
freq = Counter(s)

# Check anagram
def is_anagram(s1, s2):
    return Counter(s1) == Counter(s2)

# Generate all substrings
def all_substrings(s):
    result = []
    n = len(s)
    for i in range(n):
        for j in range(i + 1, n + 1):
            result.append(s[i:j])
    return result
```

### 6.3 Longest Palindromic Substring

```python
"""
Problem: Find the longest palindromic substring.
Input: s = "babad"
Output: "bab" or "aba"
"""

def longest_palindrome(s):
    """
    Expand around center for each possible center.
    
    Time: O(n²), Space: O(1)
    """
    if not s:
        return ""
    
    def expand_around_center(left, right):
        """Expand outward while palindrome."""
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return s[left + 1:right]
    
    result = ""
    for i in range(len(s)):
        # Odd length palindrome (single center)
        odd = expand_around_center(i, i)
        if len(odd) > len(result):
            result = odd
        
        # Even length palindrome (double center)
        even = expand_around_center(i, i + 1)
        if len(even) > len(result):
            result = even
    
    return result
```

### 6.4 String Compression

```python
"""
Problem: Compress string using counts.
Input: chars = ["a","a","b","b","c","c","c"]
Output: 6 (["a","2","b","2","c","3"])
"""

def compress(chars):
    """
    In-place compression using read/write pointers.
    
    Time: O(n), Space: O(1)
    """
    write = 0
    read = 0
    n = len(chars)
    
    while read < n:
        char = chars[read]
        count = 0
        
        # Count consecutive characters
        while read < n and chars[read] == char:
            read += 1
            count += 1
        
        # Write character
        chars[write] = char
        write += 1
        
        # Write count if > 1
        if count > 1:
            for digit in str(count):
                chars[write] = digit
                write += 1
    
    return write
```

---

## 7. Practice Problems

### Difficulty Progression

```
EASY (Start Here):
├── Two Sum II - Sorted (Two Pointers)
├── Valid Palindrome (Two Pointers)
├── Maximum Subarray (Kadane's)
├── Best Time to Buy/Sell Stock
├── Contains Duplicate
├── Binary Search
├── First Bad Version

MEDIUM (Core Problems):
├── 3Sum (Two Pointers)
├── Container With Most Water
├── Longest Substring Without Repeating
├── Minimum Window Substring
├── Product of Array Except Self
├── Search in Rotated Sorted Array
├── Find Peak Element
├── Longest Palindromic Substring

HARD (Challenge):
├── Median of Two Sorted Arrays
├── Minimum Window Substring
├── Longest Valid Parentheses
├── Trapping Rain Water
```

### LeetCode Problem List

```
ARRAYS:
1. Two Sum (Easy)
15. 3Sum (Medium)
11. Container With Most Water (Medium)
42. Trapping Rain Water (Hard)
53. Maximum Subarray (Medium)
121. Best Time to Buy/Sell Stock (Easy)
152. Maximum Product Subarray (Medium)
238. Product of Array Except Self (Medium)
287. Find the Duplicate Number (Medium)

TWO POINTERS:
167. Two Sum II (Medium)
125. Valid Palindrome (Easy)
15. 3Sum (Medium)
26. Remove Duplicates (Easy)
27. Remove Element (Easy)
283. Move Zeroes (Easy)

SLIDING WINDOW:
3. Longest Substring Without Repeating (Medium)
76. Minimum Window Substring (Hard)
209. Minimum Size Subarray Sum (Medium)
424. Longest Repeating Character Replacement (Medium)
567. Permutation in String (Medium)

BINARY SEARCH:
704. Binary Search (Easy)
33. Search in Rotated Array (Medium)
34. Find First and Last Position (Medium)
153. Find Minimum in Rotated Array (Medium)
162. Find Peak Element (Medium)
875. Koko Eating Bananas (Medium)

STRINGS:
5. Longest Palindromic Substring (Medium)
49. Group Anagrams (Medium)
242. Valid Anagram (Easy)
647. Palindromic Substrings (Medium)
```

---

## Summary

Key patterns for arrays and strings:

1. **Two Pointers**: Converging or same direction for pair/subset problems
2. **Sliding Window**: Fixed or variable window for subarray problems
3. **Binary Search**: O(log n) search on sorted data or answers
4. **Prefix Sum**: Precompute cumulative sums for range queries
5. **String Manipulation**: Know common operations and patterns

Master these patterns, and you'll solve 60%+ of array/string problems.

---

**Next Part**: [Part 3: Hash Maps & Sets](./DSA-Complete-Guide-Part3-HashMaps-Sets.md)
