# Complete DSA Guide - Crack Any Coding Interview
# Part 5: Trees & Binary Search Trees

---

## Table of Contents

1. [Tree Fundamentals](#1-tree-fundamentals)
2. [Tree Traversals](#2-tree-traversals)
3. [Binary Tree Patterns](#3-binary-tree-patterns)
4. [Binary Search Tree](#4-binary-search-tree)
5. [Tree Construction](#5-tree-construction)
6. [Practice Problems](#6-practice-problems)

---

## 1. Tree Fundamentals

### 1.1 Tree Terminology

```
Tree Structure:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                         [1] ← Root                              │
│                        /   \                                    │
│                      [2]   [3] ← Children of 1                  │
│                      / \     \                                  │
│                    [4] [5]   [6] ← Leaves (no children)         │
│                                                                 │
│  Terminology:                                                   │
│  ├── Root: Top node (1)                                         │
│  ├── Parent: Node above (2 is parent of 4, 5)                   │
│  ├── Child: Node below (4, 5 are children of 2)                 │
│  ├── Siblings: Same parent (4, 5 are siblings)                  │
│  ├── Leaf: No children (4, 5, 6)                                │
│  ├── Height: Longest path from root to leaf (2 for this tree)  │
│  ├── Depth: Distance from root (root has depth 0)              │
│  └── Subtree: Tree formed by any node and its descendants       │
│                                                                 │
│  Binary Tree: Each node has at most 2 children                  │
│  Full Binary Tree: Every node has 0 or 2 children               │
│  Complete Binary Tree: All levels filled except possibly last   │
│  Perfect Binary Tree: All levels completely filled              │
│  Balanced Binary Tree: Height difference ≤ 1 for all nodes      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Node Definition

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Creating a tree
#       1
#      / \
#     2   3
#    / \
#   4   5

root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)
```

---

## 2. Tree Traversals

### 2.1 DFS Traversals (Recursive)

```python
def preorder(root):
    """Root → Left → Right"""
    if not root:
        return []
    return [root.val] + preorder(root.left) + preorder(root.right)


def inorder(root):
    """Left → Root → Right (sorted for BST!)"""
    if not root:
        return []
    return inorder(root.left) + [root.val] + inorder(root.right)


def postorder(root):
    """Left → Right → Root"""
    if not root:
        return []
    return postorder(root.left) + postorder(root.right) + [root.val]


# Example tree:
#       1
#      / \
#     2   3
#    / \
#   4   5
# Preorder:  [1, 2, 4, 5, 3]
# Inorder:   [4, 2, 5, 1, 3]
# Postorder: [4, 5, 2, 3, 1]
```

### 2.2 DFS Traversals (Iterative)

```python
def preorder_iterative(root):
    """Iterative preorder using stack."""
    if not root:
        return []
    
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)
        
        # Push right first so left is processed first
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
    
    return result


def inorder_iterative(root):
    """Iterative inorder using stack."""
    result = []
    stack = []
    current = root
    
    while current or stack:
        # Go left as far as possible
        while current:
            stack.append(current)
            current = current.left
        
        # Process node
        current = stack.pop()
        result.append(current.val)
        
        # Go right
        current = current.right
    
    return result


def postorder_iterative(root):
    """Iterative postorder using two stacks."""
    if not root:
        return []
    
    stack1 = [root]
    stack2 = []
    
    while stack1:
        node = stack1.pop()
        stack2.append(node.val)
        if node.left:
            stack1.append(node.left)
        if node.right:
            stack1.append(node.right)
    
    return stack2[::-1]
```

### 2.3 BFS (Level Order)

```python
from collections import deque

def level_order(root):
    """
    BFS traversal - level by level.
    
    Time: O(n), Space: O(n)
    """
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        level = []
        level_size = len(queue)
        
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        
        result.append(level)
    
    return result

# Example: [[1], [2, 3], [4, 5]]
```

---

## 3. Binary Tree Patterns

### 3.1 Maximum Depth

```python
"""
Problem: Find maximum depth of binary tree.
"""

def max_depth(root):
    """
    Recursive: depth = 1 + max(left depth, right depth)
    
    Time: O(n), Space: O(h) where h = height
    """
    if not root:
        return 0
    
    return 1 + max(max_depth(root.left), max_depth(root.right))


def max_depth_bfs(root):
    """BFS approach - count levels."""
    if not root:
        return 0
    
    depth = 0
    queue = deque([root])
    
    while queue:
        depth += 1
        for _ in range(len(queue)):
            node = queue.popleft()
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
    
    return depth
```

### 3.2 Same Tree / Symmetric Tree

```python
def is_same_tree(p, q):
    """Check if two trees are identical."""
    if not p and not q:
        return True
    if not p or not q:
        return False
    
    return (p.val == q.val and
            is_same_tree(p.left, q.left) and
            is_same_tree(p.right, q.right))


def is_symmetric(root):
    """Check if tree is symmetric (mirror of itself)."""
    def is_mirror(left, right):
        if not left and not right:
            return True
        if not left or not right:
            return False
        
        return (left.val == right.val and
                is_mirror(left.left, right.right) and
                is_mirror(left.right, right.left))
    
    return is_mirror(root, root)
```

### 3.3 Invert Binary Tree

```python
"""
Problem: Mirror/invert a binary tree.
"""

def invert_tree(root):
    """
    Swap left and right children recursively.
    
    Time: O(n), Space: O(h)
    """
    if not root:
        return None
    
    # Swap children
    root.left, root.right = root.right, root.left
    
    # Recurse
    invert_tree(root.left)
    invert_tree(root.right)
    
    return root
```

### 3.4 Path Sum

```python
"""
Problem: Check if path from root to leaf sums to target.
"""

def has_path_sum(root, target_sum):
    """
    Time: O(n), Space: O(h)
    """
    if not root:
        return False
    
    # Leaf node
    if not root.left and not root.right:
        return root.val == target_sum
    
    remaining = target_sum - root.val
    return (has_path_sum(root.left, remaining) or
            has_path_sum(root.right, remaining))


def path_sum_all(root, target_sum):
    """Find ALL root-to-leaf paths that sum to target."""
    result = []
    
    def dfs(node, remaining, path):
        if not node:
            return
        
        path.append(node.val)
        
        # Leaf node
        if not node.left and not node.right and remaining == node.val:
            result.append(path[:])  # Copy of path
        
        dfs(node.left, remaining - node.val, path)
        dfs(node.right, remaining - node.val, path)
        
        path.pop()  # Backtrack
    
    dfs(root, target_sum, [])
    return result
```

### 3.5 Lowest Common Ancestor

```python
"""
Problem: Find LCA of two nodes in binary tree.
"""

def lowest_common_ancestor(root, p, q):
    """
    If both nodes are found in different subtrees,
    current node is LCA.
    
    Time: O(n), Space: O(h)
    """
    if not root or root == p or root == q:
        return root
    
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    
    # If both sides found something, this is LCA
    if left and right:
        return root
    
    # Otherwise, return whichever side found something
    return left or right
```

### 3.6 Diameter of Binary Tree

```python
"""
Problem: Find longest path between any two nodes.
"""

def diameter_of_binary_tree(root):
    """
    Diameter at each node = left height + right height.
    
    Time: O(n), Space: O(h)
    """
    diameter = 0
    
    def height(node):
        nonlocal diameter
        if not node:
            return 0
        
        left_h = height(node.left)
        right_h = height(node.right)
        
        # Update diameter (path through this node)
        diameter = max(diameter, left_h + right_h)
        
        return 1 + max(left_h, right_h)
    
    height(root)
    return diameter
```

### 3.7 Binary Tree Maximum Path Sum

```python
"""
Problem: Find path with maximum sum (any path).
"""

def max_path_sum(root):
    """
    At each node, consider:
    1. Node alone
    2. Node + left path
    3. Node + right path
    4. Node + left + right (can't extend upward)
    
    Time: O(n), Space: O(h)
    """
    max_sum = float('-inf')
    
    def max_gain(node):
        nonlocal max_sum
        if not node:
            return 0
        
        # Max gain from left/right subtrees (0 if negative)
        left_gain = max(max_gain(node.left), 0)
        right_gain = max(max_gain(node.right), 0)
        
        # Path through this node
        path_sum = node.val + left_gain + right_gain
        max_sum = max(max_sum, path_sum)
        
        # Return max gain if continuing path upward
        return node.val + max(left_gain, right_gain)
    
    max_gain(root)
    return max_sum
```

---

## 4. Binary Search Tree

### 4.1 BST Properties

```
Binary Search Tree:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  For every node:                                                │
│  ├── All values in LEFT subtree < node's value                  │
│  └── All values in RIGHT subtree > node's value                 │
│                                                                 │
│         [8]                                                     │
│        /   \                                                    │
│      [3]   [10]                                                 │
│      / \      \                                                 │
│    [1] [6]   [14]                                               │
│        / \   /                                                  │
│      [4][7][13]                                                 │
│                                                                 │
│  Inorder traversal gives SORTED order!                          │
│  [1, 3, 4, 6, 7, 8, 10, 13, 14]                                  │
│                                                                 │
│  Operations (balanced BST):                                     │
│  ├── Search: O(log n)                                           │
│  ├── Insert: O(log n)                                           │
│  └── Delete: O(log n)                                           │
│                                                                 │
│  Worst case (skewed): O(n)                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 BST Search and Insert

```python
def search_bst(root, val):
    """Search for value in BST."""
    if not root or root.val == val:
        return root
    
    if val < root.val:
        return search_bst(root.left, val)
    else:
        return search_bst(root.right, val)


def insert_bst(root, val):
    """Insert value into BST."""
    if not root:
        return TreeNode(val)
    
    if val < root.val:
        root.left = insert_bst(root.left, val)
    else:
        root.right = insert_bst(root.right, val)
    
    return root
```

### 4.3 Validate BST

```python
"""
Problem: Check if tree is valid BST.
"""

def is_valid_bst(root):
    """
    Track valid range for each node.
    
    Time: O(n), Space: O(h)
    """
    def validate(node, min_val, max_val):
        if not node:
            return True
        
        if node.val <= min_val or node.val >= max_val:
            return False
        
        return (validate(node.left, min_val, node.val) and
                validate(node.right, node.val, max_val))
    
    return validate(root, float('-inf'), float('inf'))


def is_valid_bst_inorder(root):
    """Alternative: Inorder should be strictly increasing."""
    prev = float('-inf')
    
    def inorder(node):
        nonlocal prev
        if not node:
            return True
        
        if not inorder(node.left):
            return False
        
        if node.val <= prev:
            return False
        prev = node.val
        
        return inorder(node.right)
    
    return inorder(root)
```

### 4.4 Kth Smallest Element in BST

```python
"""
Problem: Find kth smallest element in BST.
"""

def kth_smallest(root, k):
    """
    Inorder traversal stops at kth element.
    
    Time: O(h + k), Space: O(h)
    """
    stack = []
    current = root
    count = 0
    
    while current or stack:
        while current:
            stack.append(current)
            current = current.left
        
        current = stack.pop()
        count += 1
        
        if count == k:
            return current.val
        
        current = current.right
    
    return -1
```

### 4.5 Delete Node in BST

```python
"""
Problem: Delete a node from BST.
"""

def delete_node(root, key):
    """
    Three cases:
    1. Node is leaf: just remove
    2. Node has one child: replace with child
    3. Node has two children: replace with inorder successor
    
    Time: O(h), Space: O(h)
    """
    if not root:
        return None
    
    if key < root.val:
        root.left = delete_node(root.left, key)
    elif key > root.val:
        root.right = delete_node(root.right, key)
    else:
        # Found node to delete
        
        # Case 1 & 2: One or no child
        if not root.left:
            return root.right
        if not root.right:
            return root.left
        
        # Case 3: Two children
        # Find inorder successor (smallest in right subtree)
        successor = root.right
        while successor.left:
            successor = successor.left
        
        root.val = successor.val
        root.right = delete_node(root.right, successor.val)
    
    return root
```

---

## 5. Tree Construction

### 5.1 Build Tree from Preorder and Inorder

```python
"""
Problem: Construct binary tree from preorder and inorder traversals.
Preorder: [3, 9, 20, 15, 7]
Inorder:  [9, 3, 15, 20, 7]
"""

def build_tree(preorder, inorder):
    """
    First element of preorder is root.
    Find root in inorder to split left/right subtrees.
    
    Time: O(n), Space: O(n)
    """
    if not preorder or not inorder:
        return None
    
    # Create index map for quick lookup
    inorder_map = {val: idx for idx, val in enumerate(inorder)}
    
    def build(pre_start, pre_end, in_start, in_end):
        if pre_start > pre_end:
            return None
        
        root_val = preorder[pre_start]
        root = TreeNode(root_val)
        
        root_idx = inorder_map[root_val]
        left_size = root_idx - in_start
        
        root.left = build(pre_start + 1, pre_start + left_size,
                          in_start, root_idx - 1)
        root.right = build(pre_start + left_size + 1, pre_end,
                           root_idx + 1, in_end)
        
        return root
    
    return build(0, len(preorder) - 1, 0, len(inorder) - 1)
```

### 5.2 Serialize and Deserialize

```python
"""
Problem: Serialize tree to string and deserialize back.
"""

class Codec:
    def serialize(self, root):
        """Preorder with 'null' for None."""
        if not root:
            return "null"
        
        return (str(root.val) + "," +
                self.serialize(root.left) + "," +
                self.serialize(root.right))
    
    def deserialize(self, data):
        """Rebuild tree from serialized string."""
        values = iter(data.split(","))
        
        def build():
            val = next(values)
            if val == "null":
                return None
            
            node = TreeNode(int(val))
            node.left = build()
            node.right = build()
            return node
        
        return build()
```

---

## 6. Practice Problems

```
EASY:
├── 104. Maximum Depth (Easy) ⭐
├── 100. Same Tree (Easy)
├── 101. Symmetric Tree (Easy) ⭐
├── 226. Invert Binary Tree (Easy) ⭐
├── 112. Path Sum (Easy)
├── 543. Diameter of Binary Tree (Easy)
├── 617. Merge Two Binary Trees (Easy)
├── 700. Search in BST (Easy)
├── 108. Convert Sorted Array to BST (Easy)

MEDIUM:
├── 102. Binary Tree Level Order (Medium) ⭐
├── 98. Validate BST (Medium) ⭐
├── 230. Kth Smallest Element in BST (Medium)
├── 236. Lowest Common Ancestor (Medium) ⭐
├── 105. Construct from Preorder/Inorder (Medium)
├── 199. Binary Tree Right Side View (Medium)
├── 113. Path Sum II (Medium)
├── 450. Delete Node in BST (Medium)

HARD:
├── 124. Binary Tree Maximum Path Sum (Hard) ⭐
├── 297. Serialize and Deserialize (Hard)
├── 968. Binary Tree Cameras (Hard)
```

---

## Summary

Tree mastery requires:

1. **Traversals**: Know preorder, inorder, postorder, level-order
2. **Recursion**: Think in terms of "solve for subtrees"
3. **BST Properties**: Inorder gives sorted order
4. **Common patterns**: Depth, paths, LCA, validation

Key insight: Most tree problems are solved by:
- Asking "what info do I need from left/right subtrees?"
- Combining that info at the current node

---

**Next Part**: [Part 6: Graphs](./DSA-Complete-Guide-Part6-Graphs.md)
