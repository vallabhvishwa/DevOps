# Complete DSA Guide - Crack Any Coding Interview
# Part 8: Advanced Data Structures

---

## Table of Contents

1. [Heaps (Priority Queues)](#1-heaps-priority-queues)
2. [Tries (Prefix Trees)](#2-tries-prefix-trees)
3. [Union-Find (Disjoint Set)](#3-union-find-disjoint-set)
4. [Segment Trees](#4-segment-trees)
5. [Practice Problems](#5-practice-problems)

---

## 1. Heaps (Priority Queues)

### 1.1 Heap Basics

```python
import heapq

# MIN HEAP (default in Python)
min_heap = []
heapq.heappush(min_heap, 3)
heapq.heappush(min_heap, 1)
heapq.heappush(min_heap, 2)
smallest = heapq.heappop(min_heap)  # Returns 1
peek = min_heap[0]                   # Peek without removing

# Heapify existing list
arr = [3, 1, 4, 1, 5]
heapq.heapify(arr)  # O(n) - converts to heap in-place

# MAX HEAP (negate values)
max_heap = []
heapq.heappush(max_heap, -3)
heapq.heappush(max_heap, -1)
largest = -heapq.heappop(max_heap)  # Returns 3

# N largest/smallest
heapq.nlargest(3, arr)   # 3 largest elements
heapq.nsmallest(3, arr)  # 3 smallest elements
```

### 1.2 Kth Largest Element

```python
"""
Problem: Find kth largest element in array.
Input: nums = [3,2,1,5,6,4], k = 2
Output: 5
"""

def find_kth_largest(nums, k):
    """
    Use min heap of size k.
    
    Time: O(n log k), Space: O(k)
    """
    heap = nums[:k]
    heapq.heapify(heap)
    
    for num in nums[k:]:
        if num > heap[0]:
            heapq.heapreplace(heap, num)
    
    return heap[0]


# Alternative: QuickSelect O(n) average
def find_kth_largest_quickselect(nums, k):
    """Partition until kth largest found."""
    k = len(nums) - k  # Convert to kth smallest index
    
    def quickselect(left, right):
        pivot = nums[right]
        p = left
        
        for i in range(left, right):
            if nums[i] <= pivot:
                nums[p], nums[i] = nums[i], nums[p]
                p += 1
        
        nums[p], nums[right] = nums[right], nums[p]
        
        if p == k:
            return nums[p]
        elif p < k:
            return quickselect(p + 1, right)
        else:
            return quickselect(left, p - 1)
    
    return quickselect(0, len(nums) - 1)
```

### 1.3 Top K Frequent Elements

```python
"""
Problem: Find k most frequent elements.
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1, 2]
"""

from collections import Counter

def top_k_frequent(nums, k):
    """
    Use heap of size k.
    
    Time: O(n log k), Space: O(n)
    """
    count = Counter(nums)
    return heapq.nlargest(k, count.keys(), key=count.get)


# Bucket sort approach: O(n)
def top_k_frequent_bucket(nums, k):
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]
    
    for num, freq in count.items():
        buckets[freq].append(num)
    
    result = []
    for i in range(len(buckets) - 1, 0, -1):
        for num in buckets[i]:
            result.append(num)
            if len(result) == k:
                return result
    
    return result
```

### 1.4 Merge K Sorted Lists

```python
"""
Problem: Merge k sorted linked lists.
"""

def merge_k_lists(lists):
    """
    Use heap to track smallest element from each list.
    
    Time: O(n log k), Space: O(k)
    """
    heap = []
    
    # Add first element from each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst.val, i, lst))
    
    dummy = ListNode(0)
    current = dummy
    
    while heap:
        val, i, node = heapq.heappop(heap)
        current.next = node
        current = current.next
        
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    
    return dummy.next
```

### 1.5 Find Median from Data Stream

```python
"""
Problem: Find median of streaming numbers.
"""

class MedianFinder:
    def __init__(self):
        self.small = []  # Max heap (left half)
        self.large = []  # Min heap (right half)
    
    def addNum(self, num):
        # Add to max heap (small)
        heapq.heappush(self.small, -num)
        
        # Ensure all elements in small <= all in large
        if self.small and self.large and -self.small[0] > self.large[0]:
            heapq.heappush(self.large, -heapq.heappop(self.small))
        
        # Balance sizes
        if len(self.small) > len(self.large) + 1:
            heapq.heappush(self.large, -heapq.heappop(self.small))
        if len(self.large) > len(self.small):
            heapq.heappush(self.small, -heapq.heappop(self.large))
    
    def findMedian(self):
        if len(self.small) > len(self.large):
            return -self.small[0]
        return (-self.small[0] + self.large[0]) / 2
```

---

## 2. Tries (Prefix Trees)

### 2.1 Trie Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        """Insert word into trie. O(m)"""
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True
    
    def search(self, word):
        """Return True if word exists. O(m)"""
        node = self._find_node(word)
        return node is not None and node.is_end
    
    def startsWith(self, prefix):
        """Return True if any word starts with prefix. O(m)"""
        return self._find_node(prefix) is not None
    
    def _find_node(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node.children:
                return None
            node = node.children[char]
        return node
```

### 2.2 Word Search II

```python
"""
Problem: Find all words from dictionary that exist in board.
"""

def find_words(board, words):
    """
    Build trie from words, DFS on board.
    
    Time: O(m*n * 4^L + W*L), Space: O(W*L)
    """
    # Build trie
    trie = Trie()
    for word in words:
        trie.insert(word)
    
    m, n = len(board), len(board[0])
    result = set()
    
    def dfs(r, c, node, path):
        if node.is_end:
            result.add(path)
        
        if r < 0 or r >= m or c < 0 or c >= n:
            return
        
        char = board[r][c]
        if char not in node.children:
            return
        
        board[r][c] = '#'  # Mark visited
        next_node = node.children[char]
        
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            dfs(r + dr, c + dc, next_node, path + char)
        
        board[r][c] = char  # Backtrack
    
    for r in range(m):
        for c in range(n):
            dfs(r, c, trie.root, "")
    
    return list(result)
```

---

## 3. Union-Find (Disjoint Set)

### 3.1 Union-Find Implementation

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.count = n
    
    def find(self, x):
        """Find with path compression."""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """Union by rank."""
        px, py = self.find(x), self.find(y)
        
        if px == py:
            return False
        
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        
        self.count -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

### 3.2 Number of Connected Components

```python
"""
Problem: Count connected components in undirected graph.
"""

def count_components(n, edges):
    uf = UnionFind(n)
    
    for u, v in edges:
        uf.union(u, v)
    
    return uf.count
```

### 3.3 Redundant Connection

```python
"""
Problem: Find edge that creates a cycle.
"""

def find_redundant_connection(edges):
    """
    Edge that fails to union creates cycle.
    """
    n = len(edges)
    uf = UnionFind(n + 1)
    
    for u, v in edges:
        if not uf.union(u, v):
            return [u, v]
    
    return []
```

---

## 4. Segment Trees

### 4.1 Basic Segment Tree

```python
class SegmentTree:
    """Range sum queries with point updates."""
    
    def __init__(self, nums):
        self.n = len(nums)
        self.tree = [0] * (2 * self.n)
        
        # Build tree
        for i in range(self.n):
            self.tree[self.n + i] = nums[i]
        for i in range(self.n - 1, 0, -1):
            self.tree[i] = self.tree[2 * i] + self.tree[2 * i + 1]
    
    def update(self, index, val):
        """Update element at index. O(log n)"""
        i = index + self.n
        self.tree[i] = val
        
        while i > 1:
            i //= 2
            self.tree[i] = self.tree[2 * i] + self.tree[2 * i + 1]
    
    def query(self, left, right):
        """Sum of range [left, right). O(log n)"""
        result = 0
        left += self.n
        right += self.n
        
        while left < right:
            if left % 2 == 1:
                result += self.tree[left]
                left += 1
            if right % 2 == 1:
                right -= 1
                result += self.tree[right]
            left //= 2
            right //= 2
        
        return result
```

---

## 5. Practice Problems

```
HEAPS:
├── 215. Kth Largest Element (Medium) ⭐
├── 347. Top K Frequent Elements (Medium) ⭐
├── 23. Merge K Sorted Lists (Hard) ⭐
├── 295. Find Median from Data Stream (Hard)
├── 373. Find K Pairs with Smallest Sums (Medium)
├── 767. Reorganize String (Medium)

TRIES:
├── 208. Implement Trie (Medium) ⭐
├── 211. Design Add and Search Words (Medium)
├── 212. Word Search II (Hard)
├── 648. Replace Words (Medium)

UNION-FIND:
├── 200. Number of Islands (Medium) - can use UF
├── 547. Number of Provinces (Medium) ⭐
├── 684. Redundant Connection (Medium)
├── 721. Accounts Merge (Medium)
├── 990. Satisfiability of Equality (Medium)
```

---

## Summary

1. **Heaps**: K-th element, top-K, streaming median
2. **Tries**: Prefix matching, autocomplete, word search
3. **Union-Find**: Connected components, cycle detection
4. **Segment Trees**: Range queries with updates

Know when to use each structure for optimal solutions.

---

**Next Part**: [Part 9: Problem-Solving Patterns](./DSA-Complete-Guide-Part9-Patterns.md)
