# Complete DSA Guide - Crack Any Coding Interview
# Part 4: Linked Lists, Stacks & Queues

---

## Table of Contents

1. [Linked List Fundamentals](#1-linked-list-fundamentals)
2. [Linked List Patterns](#2-linked-list-patterns)
3. [Stack Fundamentals](#3-stack-fundamentals)
4. [Stack Patterns](#4-stack-patterns)
5. [Queue Fundamentals](#5-queue-fundamentals)
6. [Queue Patterns](#6-queue-patterns)
7. [Practice Problems](#7-practice-problems)

---

## 1. Linked List Fundamentals

### 1.1 Node Definition

```python
class ListNode:
    """Singly Linked List Node"""
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class DoublyListNode:
    """Doubly Linked List Node"""
    def __init__(self, val=0, prev=None, next=None):
        self.val = val
        self.prev = prev
        self.next = next
```

### 1.2 Basic Operations

```python
# Creating a linked list: 1 -> 2 -> 3
head = ListNode(1)
head.next = ListNode(2)
head.next.next = ListNode(3)

# Traversal
def traverse(head):
    current = head
    while current:
        print(current.val)
        current = current.next

# Find length
def get_length(head):
    length = 0
    current = head
    while current:
        length += 1
        current = current.next
    return length

# Find node at index
def get_node(head, index):
    current = head
    for _ in range(index):
        if not current:
            return None
        current = current.next
    return current

# Insert at head
def insert_at_head(head, val):
    new_node = ListNode(val)
    new_node.next = head
    return new_node  # New head

# Insert at tail
def insert_at_tail(head, val):
    new_node = ListNode(val)
    if not head:
        return new_node
    
    current = head
    while current.next:
        current = current.next
    current.next = new_node
    return head

# Delete node with value
def delete_node(head, val):
    dummy = ListNode(0)
    dummy.next = head
    
    current = dummy
    while current.next:
        if current.next.val == val:
            current.next = current.next.next
            break
        current = current.next
    
    return dummy.next
```

### 1.3 Dummy Head Technique

```python
"""
Use a dummy node to simplify edge cases at head.
"""

def remove_elements(head, val):
    """Remove all nodes with given value."""
    dummy = ListNode(0)
    dummy.next = head
    
    current = dummy
    while current.next:
        if current.next.val == val:
            current.next = current.next.next
        else:
            current = current.next
    
    return dummy.next
```

---

## 2. Linked List Patterns

### 2.1 Reverse Linked List

```python
"""
Problem: Reverse a singly linked list.
Input: 1 -> 2 -> 3 -> 4 -> 5
Output: 5 -> 4 -> 3 -> 2 -> 1
"""

def reverse_list(head):
    """
    Iterative: Use three pointers.
    
    Time: O(n), Space: O(1)
    """
    prev = None
    current = head
    
    while current:
        next_temp = current.next  # Save next
        current.next = prev       # Reverse pointer
        prev = current            # Move prev forward
        current = next_temp       # Move current forward
    
    return prev


def reverse_list_recursive(head):
    """
    Recursive approach.
    
    Time: O(n), Space: O(n) - recursion stack
    """
    if not head or not head.next:
        return head
    
    new_head = reverse_list_recursive(head.next)
    head.next.next = head
    head.next = None
    
    return new_head
```

### 2.2 Fast and Slow Pointers

```python
"""
Pattern: Use two pointers moving at different speeds.
"""

# Find middle of linked list
def find_middle(head):
    """
    Slow moves 1 step, fast moves 2 steps.
    When fast reaches end, slow is at middle.
    
    Time: O(n), Space: O(1)
    """
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    return slow


# Detect cycle
def has_cycle(head):
    """
    If there's a cycle, fast will eventually meet slow.
    
    Time: O(n), Space: O(1)
    """
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        
        if slow == fast:
            return True
    
    return False


# Find cycle start
def detect_cycle(head):
    """
    After detecting cycle, reset one pointer to head.
    Move both at same speed - they meet at cycle start.
    
    Time: O(n), Space: O(1)
    """
    slow = fast = head
    
    # Find meeting point
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        
        if slow == fast:
            # Reset slow to head
            slow = head
            # Move both at same speed
            while slow != fast:
                slow = slow.next
                fast = fast.next
            return slow
    
    return None
```

### 2.3 Merge Two Sorted Lists

```python
"""
Problem: Merge two sorted linked lists.
Input: l1 = 1->2->4, l2 = 1->3->4
Output: 1->1->2->3->4->4
"""

def merge_two_lists(l1, l2):
    """
    Use dummy node and compare values.
    
    Time: O(n + m), Space: O(1)
    """
    dummy = ListNode(0)
    current = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next
    
    # Attach remaining nodes
    current.next = l1 or l2
    
    return dummy.next
```

### 2.4 Remove Nth Node From End

```python
"""
Problem: Remove nth node from end.
Input: head = 1->2->3->4->5, n = 2
Output: 1->2->3->5
"""

def remove_nth_from_end(head, n):
    """
    Use two pointers n nodes apart.
    
    Time: O(n), Space: O(1)
    """
    dummy = ListNode(0)
    dummy.next = head
    
    # Move fast n+1 steps ahead
    fast = dummy
    for _ in range(n + 1):
        fast = fast.next
    
    # Move both until fast reaches end
    slow = dummy
    while fast:
        slow = slow.next
        fast = fast.next
    
    # Remove nth node
    slow.next = slow.next.next
    
    return dummy.next
```

### 2.5 Palindrome Linked List

```python
"""
Problem: Check if linked list is palindrome.
Input: 1 -> 2 -> 2 -> 1
Output: True
"""

def is_palindrome(head):
    """
    Find middle, reverse second half, compare.
    
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return True
    
    # Find middle
    slow = fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    
    # Reverse second half
    second_half = reverse_list(slow.next)
    
    # Compare halves
    first_half = head
    while second_half:
        if first_half.val != second_half.val:
            return False
        first_half = first_half.next
        second_half = second_half.next
    
    return True
```

---

## 3. Stack Fundamentals

### 3.1 Stack Basics

```python
"""
Stack: LIFO (Last In, First Out)
"""

# Using list
stack = []
stack.append(1)     # Push
stack.append(2)
stack.pop()         # Pop (returns 2)
stack[-1]           # Peek (top element)
len(stack)          # Size
not stack           # Is empty?

# Using deque (slightly faster)
from collections import deque
stack = deque()
stack.append(1)     # Push
stack.pop()         # Pop
stack[-1]           # Peek
```

### 3.2 Valid Parentheses

```python
"""
Problem: Check if parentheses are valid.
Input: s = "()[]{}"
Output: True
"""

def is_valid(s):
    """
    Use stack to match opening and closing brackets.
    
    Time: O(n), Space: O(n)
    """
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            # Closing bracket
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            # Opening bracket
            stack.append(char)
    
    return len(stack) == 0
```

---

## 4. Stack Patterns

### 4.1 Monotonic Stack

```python
"""
Monotonic Stack: Stack that maintains elements in sorted order.
Used for "next greater/smaller element" problems.
"""

# Next Greater Element
def next_greater_element(nums):
    """
    For each element, find next greater element to its right.
    
    Time: O(n), Space: O(n)
    """
    result = [-1] * len(nums)
    stack = []  # Store indices
    
    for i in range(len(nums)):
        # Pop elements smaller than current
        while stack and nums[stack[-1]] < nums[i]:
            idx = stack.pop()
            result[idx] = nums[i]
        stack.append(i)
    
    return result

# Example: [4, 5, 2, 25] → [5, 25, 25, -1]


# Daily Temperatures
def daily_temperatures(temperatures):
    """
    How many days until warmer temperature?
    
    Time: O(n), Space: O(n)
    """
    n = len(temperatures)
    result = [0] * n
    stack = []  # Store indices
    
    for i in range(n):
        while stack and temperatures[stack[-1]] < temperatures[i]:
            idx = stack.pop()
            result[idx] = i - idx
        stack.append(i)
    
    return result
```

### 4.2 Min Stack

```python
"""
Problem: Design stack that supports getMin in O(1).
"""

class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []
    
    def push(self, val):
        self.stack.append(val)
        # Track minimum at each level
        if not self.min_stack or val <= self.min_stack[-1]:
            self.min_stack.append(val)
    
    def pop(self):
        val = self.stack.pop()
        if val == self.min_stack[-1]:
            self.min_stack.pop()
    
    def top(self):
        return self.stack[-1]
    
    def getMin(self):
        return self.min_stack[-1]
```

### 4.3 Evaluate Reverse Polish Notation

```python
"""
Problem: Evaluate expression in RPN.
Input: ["2", "1", "+", "3", "*"]
Output: 9 ((2 + 1) * 3)
"""

def eval_rpn(tokens):
    """
    Use stack for operands, evaluate when operator found.
    
    Time: O(n), Space: O(n)
    """
    stack = []
    operators = {'+', '-', '*', '/'}
    
    for token in tokens:
        if token in operators:
            b = stack.pop()
            a = stack.pop()
            if token == '+':
                stack.append(a + b)
            elif token == '-':
                stack.append(a - b)
            elif token == '*':
                stack.append(a * b)
            else:
                stack.append(int(a / b))  # Truncate towards zero
        else:
            stack.append(int(token))
    
    return stack[0]
```

---

## 5. Queue Fundamentals

### 5.1 Queue Basics

```python
"""
Queue: FIFO (First In, First Out)
"""

from collections import deque

queue = deque()
queue.append(1)      # Enqueue (right)
queue.append(2)
queue.popleft()      # Dequeue (left) - returns 1
queue[0]             # Peek front
len(queue)           # Size
```

### 5.2 Implement Queue using Stacks

```python
"""
Problem: Implement queue using only stacks.
"""

class MyQueue:
    def __init__(self):
        self.stack_in = []   # For push
        self.stack_out = []  # For pop
    
    def push(self, x):
        self.stack_in.append(x)
    
    def pop(self):
        self._move_if_needed()
        return self.stack_out.pop()
    
    def peek(self):
        self._move_if_needed()
        return self.stack_out[-1]
    
    def empty(self):
        return not self.stack_in and not self.stack_out
    
    def _move_if_needed(self):
        if not self.stack_out:
            while self.stack_in:
                self.stack_out.append(self.stack_in.pop())
```

---

## 6. Queue Patterns

### 6.1 BFS Template

```python
from collections import deque

def bfs_template(start):
    """BFS using queue for level-order traversal."""
    queue = deque([start])
    visited = {start}
    
    while queue:
        node = queue.popleft()
        # Process node
        
        for neighbor in get_neighbors(node):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

### 6.2 Sliding Window Maximum

```python
"""
Problem: Find max in each sliding window of size k.
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
"""

def max_sliding_window(nums, k):
    """
    Use monotonic deque to track max.
    
    Time: O(n), Space: O(k)
    """
    from collections import deque
    
    result = []
    dq = deque()  # Store indices, values are decreasing
    
    for i in range(len(nums)):
        # Remove elements outside window
        while dq and dq[0] < i - k + 1:
            dq.popleft()
        
        # Remove smaller elements (they can't be max)
        while dq and nums[dq[-1]] < nums[i]:
            dq.pop()
        
        dq.append(i)
        
        # Add to result once window is full
        if i >= k - 1:
            result.append(nums[dq[0]])
    
    return result
```

---

## 7. Practice Problems

```
LINKED LISTS:
├── 206. Reverse Linked List (Easy) ⭐
├── 21. Merge Two Sorted Lists (Easy) ⭐
├── 141. Linked List Cycle (Easy) ⭐
├── 142. Linked List Cycle II (Medium)
├── 19. Remove Nth From End (Medium)
├── 234. Palindrome Linked List (Easy)
├── 143. Reorder List (Medium)
├── 148. Sort List (Medium)
├── 23. Merge K Sorted Lists (Hard)

STACKS:
├── 20. Valid Parentheses (Easy) ⭐
├── 155. Min Stack (Medium)
├── 739. Daily Temperatures (Medium) ⭐
├── 150. Evaluate RPN (Medium)
├── 84. Largest Rectangle in Histogram (Hard)
├── 42. Trapping Rain Water (Hard)

QUEUES:
├── 232. Implement Queue using Stacks (Easy)
├── 225. Implement Stack using Queues (Easy)
├── 239. Sliding Window Maximum (Hard)
```

---

## Summary

1. **Linked Lists**: Master reversal, fast/slow pointers, dummy head
2. **Stacks**: LIFO for matching, expression evaluation, monotonic patterns
3. **Queues**: FIFO for BFS, sliding window with deque

Key techniques:
- **Dummy head** simplifies linked list edge cases
- **Fast/slow pointers** for cycle detection and middle finding
- **Monotonic stack** for next greater/smaller element
- **Monotonic deque** for sliding window extrema

---

**Next Part**: [Part 5: Trees](./DSA-Complete-Guide-Part5-Trees.md)
