# Complete DSA Guide - Crack Any Coding Interview
# Part 3: Hash Maps & Sets

---

## Table of Contents

1. [Hash Table Fundamentals](#1-hash-table-fundamentals)
2. [Hash Map Patterns](#2-hash-map-patterns)
3. [Hash Set Patterns](#3-hash-set-patterns)
4. [Counter Pattern](#4-counter-pattern)
5. [Common Problems](#5-common-problems)
6. [Practice Problems](#6-practice-problems)

---

## 1. Hash Table Fundamentals

### 1.1 How Hash Tables Work

```
Hash Table Internals:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Hash Function: Converts key to index                           │
│                                                                 │
│  "apple" ──► hash("apple") ──► 42 ──► bucket[42]                │
│                                                                 │
│  Buckets (Array):                                               │
│  [0] → None                                                     │
│  [1] → ("banana", 5)                                            │
│  [2] → None                                                     │
│  ...                                                            │
│  [42] → ("apple", 3) → ("apricot", 7)  ← Collision (chaining)   │
│  ...                                                            │
│                                                                 │
│  COLLISION HANDLING:                                            │
│  1. Chaining: Store collisions in linked list                   │
│  2. Open Addressing: Find next empty slot                       │
│                                                                 │
│  TIME COMPLEXITY:                                               │
│  ├── Average: O(1) for insert, lookup, delete                   │
│  └── Worst (all collisions): O(n)                               │
│                                                                 │
│  SPACE COMPLEXITY: O(n)                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Python Dictionary (Hash Map)

```python
# Creating dictionaries
d = {}                              # Empty dict
d = {'a': 1, 'b': 2}                # With initial values
d = dict(a=1, b=2)                  # Using dict()
d = {x: x**2 for x in range(5)}     # Dict comprehension

# Basic operations - All O(1) average
d[key] = value          # Insert/Update
value = d[key]          # Access (KeyError if missing)
value = d.get(key)      # Access (None if missing)
value = d.get(key, 0)   # Access with default
del d[key]              # Delete
key in d                # Check existence
len(d)                  # Size

# Common methods
d.keys()                # All keys (view)
d.values()              # All values (view)
d.items()               # All (key, value) pairs
d.pop(key)              # Remove and return value
d.pop(key, default)     # Remove or return default
d.setdefault(key, val)  # Get or set default
d.update(other_dict)    # Merge another dict

# Iteration
for key in d:                   # Iterate keys
    print(key, d[key])

for key, value in d.items():    # Iterate key-value pairs
    print(key, value)

# Default values with defaultdict
from collections import defaultdict

# Auto-initialize missing keys
d = defaultdict(int)        # Default 0
d = defaultdict(list)       # Default []
d = defaultdict(set)        # Default set()

d['count'] += 1             # Works even if 'count' doesn't exist
d['items'].append('a')      # Works even if 'items' doesn't exist
```

### 1.3 Python Set

```python
# Creating sets
s = set()                       # Empty set
s = {1, 2, 3}                   # With initial values
s = set([1, 2, 2, 3])           # From list (removes duplicates)
s = {x for x in range(5)}       # Set comprehension

# Basic operations - All O(1) average
s.add(element)          # Add element
s.remove(element)       # Remove (KeyError if missing)
s.discard(element)      # Remove (no error if missing)
element in s            # Check existence
len(s)                  # Size

# Set operations - O(min(len(s1), len(s2))) to O(len(s1) + len(s2))
s1 & s2                 # Intersection (elements in both)
s1 | s2                 # Union (elements in either)
s1 - s2                 # Difference (in s1 but not s2)
s1 ^ s2                 # Symmetric difference (in one but not both)
s1.issubset(s2)         # Is s1 ⊆ s2?
s1.issuperset(s2)       # Is s1 ⊇ s2?
s1.isdisjoint(s2)       # No common elements?

# Frozen set (immutable, hashable)
fs = frozenset([1, 2, 3])
```

---

## 2. Hash Map Patterns

### 2.1 Two Sum (Classic)

```python
"""
Problem: Find two numbers that sum to target.
Input: nums = [2, 7, 11, 15], target = 9
Output: [0, 1] (indices of 2 and 7)
"""

def two_sum(nums, target):
    """
    For each number, check if complement (target - num) was seen.
    
    Time: O(n), Space: O(n)
    """
    seen = {}  # value -> index
    
    for i, num in enumerate(nums):
        complement = target - num
        
        if complement in seen:
            return [seen[complement], i]
        
        seen[num] = i
    
    return []


# Walkthrough for nums = [2, 7, 11, 15], target = 9:
# i=0, num=2: complement=7, not in seen, add {2: 0}
# i=1, num=7: complement=2, 2 in seen! return [0, 1]
```

### 2.2 Frequency Map Pattern

```python
"""
Use hash map to count occurrences.
"""

def count_frequency(arr):
    """Count frequency of each element."""
    freq = {}
    for num in arr:
        freq[num] = freq.get(num, 0) + 1
    return freq


# Using Counter (cleaner)
from collections import Counter

freq = Counter(arr)
freq.most_common(3)     # Top 3 most common
freq['a']               # Count of 'a' (0 if missing)


# Example: Find majority element (appears > n/2 times)
def majority_element(nums):
    """
    Time: O(n), Space: O(n)
    """
    counts = Counter(nums)
    return counts.most_common(1)[0][0]


# Boyer-Moore Voting (O(1) space alternative)
def majority_element_optimal(nums):
    candidate = None
    count = 0
    
    for num in nums:
        if count == 0:
            candidate = num
        count += 1 if num == candidate else -1
    
    return candidate
```

### 2.3 Group By Pattern

```python
"""
Problem: Group anagrams together.
Input: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
Output: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
"""

def group_anagrams(strs):
    """
    Use sorted string as key to group anagrams.
    
    Time: O(n * k log k) where k = max string length
    Space: O(n * k)
    """
    groups = defaultdict(list)
    
    for s in strs:
        # Sort letters as key (anagrams have same sorted form)
        key = ''.join(sorted(s))
        groups[key].append(s)
    
    return list(groups.values())


# Alternative: Use character count tuple as key
def group_anagrams_v2(strs):
    """O(n * k) time using count as key."""
    groups = defaultdict(list)
    
    for s in strs:
        # Count each character
        count = [0] * 26
        for c in s:
            count[ord(c) - ord('a')] += 1
        groups[tuple(count)].append(s)
    
    return list(groups.values())
```

### 2.4 Index/Position Map

```python
"""
Store positions for quick lookup.
"""

# First non-repeating character
def first_uniq_char(s):
    """
    Find index of first non-repeating character.
    Input: s = "leetcode"
    Output: 0 (character 'l')
    """
    count = Counter(s)
    
    for i, char in enumerate(s):
        if count[char] == 1:
            return i
    
    return -1


# Contains nearby duplicate
def contains_nearby_duplicate(nums, k):
    """
    Check if nums[i] == nums[j] and abs(i - j) <= k.
    """
    last_index = {}
    
    for i, num in enumerate(nums):
        if num in last_index and i - last_index[num] <= k:
            return True
        last_index[num] = i
    
    return False
```

---

## 3. Hash Set Patterns

### 3.1 Existence Check

```python
# Check if element exists in O(1)
def contains_duplicate(nums):
    """
    Check if any duplicates exist.
    """
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False


# One-liner version
def contains_duplicate_v2(nums):
    return len(nums) != len(set(nums))
```

### 3.2 Intersection and Union

```python
def intersection(nums1, nums2):
    """Find common elements."""
    return list(set(nums1) & set(nums2))


def union(nums1, nums2):
    """Find all unique elements."""
    return list(set(nums1) | set(nums2))


def difference(nums1, nums2):
    """Elements in nums1 but not nums2."""
    return list(set(nums1) - set(nums2))
```

### 3.3 Longest Consecutive Sequence

```python
"""
Problem: Find length of longest consecutive sequence.
Input: nums = [100, 4, 200, 1, 3, 2]
Output: 4 (sequence [1, 2, 3, 4])
"""

def longest_consecutive(nums):
    """
    Use set for O(1) lookup. Start counting from sequence starts.
    
    Time: O(n), Space: O(n)
    """
    num_set = set(nums)
    longest = 0
    
    for num in num_set:
        # Only start if this is the beginning of a sequence
        if num - 1 not in num_set:
            current = num
            length = 1
            
            # Count consecutive numbers
            while current + 1 in num_set:
                current += 1
                length += 1
            
            longest = max(longest, length)
    
    return longest


# Walkthrough for [100, 4, 200, 1, 3, 2]:
# num_set = {100, 4, 200, 1, 3, 2}
# 100: 99 not in set, start sequence: 100 → length 1
# 4: 3 in set, skip (not a start)
# 200: 199 not in set, start sequence: 200 → length 1
# 1: 0 not in set, start sequence: 1, 2, 3, 4 → length 4
# 3: 2 in set, skip
# 2: 1 in set, skip
# Result: 4
```

### 3.4 Happy Number

```python
"""
Problem: Determine if a number is "happy".
A happy number eventually reaches 1 when repeatedly
replacing it with sum of squares of its digits.

Input: n = 19
Output: True (19 → 82 → 68 → 100 → 1)
"""

def is_happy(n):
    """
    Use set to detect cycles.
    
    Time: O(log n), Space: O(log n)
    """
    def sum_of_squares(num):
        total = 0
        while num > 0:
            digit = num % 10
            total += digit * digit
            num //= 10
        return total
    
    seen = set()
    
    while n != 1 and n not in seen:
        seen.add(n)
        n = sum_of_squares(n)
    
    return n == 1


# Alternative: Floyd's cycle detection (O(1) space)
def is_happy_v2(n):
    def sum_of_squares(num):
        total = 0
        while num > 0:
            digit = num % 10
            total += digit * digit
            num //= 10
        return total
    
    slow = n
    fast = sum_of_squares(n)
    
    while fast != 1 and slow != fast:
        slow = sum_of_squares(slow)
        fast = sum_of_squares(sum_of_squares(fast))
    
    return fast == 1
```

---

## 4. Counter Pattern

### 4.1 Valid Anagram

```python
"""
Problem: Check if t is anagram of s.
Input: s = "anagram", t = "nagaram"
Output: True
"""

def is_anagram(s, t):
    """
    Compare character counts.
    
    Time: O(n), Space: O(1) - fixed alphabet size
    """
    if len(s) != len(t):
        return False
    
    return Counter(s) == Counter(t)


# Without Counter
def is_anagram_v2(s, t):
    if len(s) != len(t):
        return False
    
    count = [0] * 26
    
    for i in range(len(s)):
        count[ord(s[i]) - ord('a')] += 1
        count[ord(t[i]) - ord('a')] -= 1
    
    return all(c == 0 for c in count)
```

### 4.2 Ransom Note

```python
"""
Problem: Can ransomNote be constructed from magazine?
Input: ransomNote = "aa", magazine = "aab"
Output: True
"""

def can_construct(ransomNote, magazine):
    """
    Check if magazine has enough of each character.
    
    Time: O(m + n), Space: O(1)
    """
    mag_count = Counter(magazine)
    
    for char in ransomNote:
        if mag_count[char] <= 0:
            return False
        mag_count[char] -= 1
    
    return True


# Using Counter subtraction
def can_construct_v2(ransomNote, magazine):
    return not Counter(ransomNote) - Counter(magazine)
```

### 4.3 Top K Frequent Elements

```python
"""
Problem: Find k most frequent elements.
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1, 2]
"""

def top_k_frequent(nums, k):
    """
    Count frequencies, return top k.
    
    Time: O(n), Space: O(n)
    """
    count = Counter(nums)
    return [x for x, _ in count.most_common(k)]


# Using bucket sort (O(n) guaranteed)
def top_k_frequent_bucket(nums, k):
    count = Counter(nums)
    n = len(nums)
    
    # Bucket by frequency
    buckets = [[] for _ in range(n + 1)]
    for num, freq in count.items():
        buckets[freq].append(num)
    
    # Collect top k from highest frequency
    result = []
    for i in range(n, 0, -1):
        for num in buckets[i]:
            result.append(num)
            if len(result) == k:
                return result
    
    return result
```

---

## 5. Common Problems

### 5.1 Subarray Sum Equals K

```python
"""
Problem: Count subarrays that sum to k.
Input: nums = [1, 2, 3], k = 3
Output: 2 ([1, 2] and [3])
"""

def subarray_sum(nums, k):
    """
    Use prefix sum with hash map.
    
    If prefix[j] - prefix[i] = k, then sum(i+1 to j) = k.
    
    Time: O(n), Space: O(n)
    """
    count = 0
    prefix_sum = 0
    prefix_counts = {0: 1}  # Empty prefix has sum 0
    
    for num in nums:
        prefix_sum += num
        
        # How many prefix sums equal (prefix_sum - k)?
        if prefix_sum - k in prefix_counts:
            count += prefix_counts[prefix_sum - k]
        
        prefix_counts[prefix_sum] = prefix_counts.get(prefix_sum, 0) + 1
    
    return count
```

### 5.2 Isomorphic Strings

```python
"""
Problem: Check if s and t are isomorphic.
Two strings are isomorphic if characters in s can be replaced to get t.

Input: s = "egg", t = "add"
Output: True (e→a, g→d)
"""

def is_isomorphic(s, t):
    """
    Track bidirectional mapping.
    
    Time: O(n), Space: O(n)
    """
    if len(s) != len(t):
        return False
    
    s_to_t = {}
    t_to_s = {}
    
    for cs, ct in zip(s, t):
        if cs in s_to_t:
            if s_to_t[cs] != ct:
                return False
        else:
            s_to_t[cs] = ct
        
        if ct in t_to_s:
            if t_to_s[ct] != cs:
                return False
        else:
            t_to_s[ct] = cs
    
    return True
```

### 5.3 LRU Cache

```python
"""
Problem: Implement Least Recently Used cache.
"""

from collections import OrderedDict

class LRUCache:
    """
    LRU Cache using OrderedDict.
    
    All operations: O(1)
    """
    
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key not in self.cache:
            return -1
        
        # Move to end (most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        
        self.cache[key] = value
        
        if len(self.cache) > self.capacity:
            # Remove oldest (first item)
            self.cache.popitem(last=False)


# Manual implementation with doubly linked list
class Node:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCacheManual:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}  # key -> Node
        
        # Dummy head and tail
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def _add_to_head(self, node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node
    
    def get(self, key):
        if key not in self.cache:
            return -1
        
        node = self.cache[key]
        self._remove(node)
        self._add_to_head(node)
        return node.val
    
    def put(self, key, value):
        if key in self.cache:
            node = self.cache[key]
            node.val = value
            self._remove(node)
            self._add_to_head(node)
        else:
            if len(self.cache) >= self.capacity:
                # Remove LRU (tail.prev)
                lru = self.tail.prev
                self._remove(lru)
                del self.cache[lru.key]
            
            node = Node(key, value)
            self.cache[key] = node
            self._add_to_head(node)
```

---

## 6. Practice Problems

```
EASY:
├── 1. Two Sum
├── 217. Contains Duplicate
├── 242. Valid Anagram
├── 383. Ransom Note
├── 387. First Unique Character
├── 205. Isomorphic Strings
├── 290. Word Pattern
├── 349. Intersection of Two Arrays

MEDIUM:
├── 49. Group Anagrams
├── 128. Longest Consecutive Sequence
├── 347. Top K Frequent Elements
├── 560. Subarray Sum Equals K
├── 146. LRU Cache
├── 380. Insert Delete GetRandom O(1)
├── 454. 4Sum II
├── 438. Find All Anagrams in String

HARD:
├── 76. Minimum Window Substring (uses hash map)
├── 460. LFU Cache
├── 895. Maximum Frequency Stack
```

---

## Summary

Hash maps and sets provide O(1) average-case operations:

1. **Two Sum Pattern**: Check if complement exists
2. **Frequency Map**: Count occurrences, find majority
3. **Group By**: Use computed key to group elements
4. **Index Map**: Store positions for quick lookup
5. **Set Operations**: Membership, intersection, union
6. **Counter**: Compare frequencies between collections

Hash tables are the most important data structure for interviews. When stuck, ask yourself: "Can I use a hash map to speed this up?"

---

**Next Part**: [Part 4: Linked Lists, Stacks & Queues](./DSA-Complete-Guide-Part4-LinkedLists-Stacks-Queues.md)
