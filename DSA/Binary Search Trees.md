---
title: Binary Search Trees, Balanced BSTs & Range Validation
aliases:
  - Binary Search Trees
  - BST
  - AVL Tree
  - Red-Black Tree
  - BST Validation
tags:
  - dsa
  - trees
  - bst
status: complete
---

# Binary Search Trees, Balanced BSTs & Range Validation

## TL;DR

A **Binary Search Tree (BST)** is a node-based binary tree data structure with the strict **BST Ordering Invariant**:
- The value of all nodes in the **left subtree** is strictly **less than** the root's value.
- The value of all nodes in the **right subtree** is strictly **greater than** the root's value.
- Both left and right subtrees must also be binary search trees.

This property enables $O(H)$ time complexity for search, insertion, and deletion, where $H$ is the tree height ($H = O(\log N)$ in balanced trees, $H = O(N)$ in degenerate skewed trees).

---

## 1. The Core BST Invariant & Inorder Traversal

```text
               8 (Root)
             /   \
            3     10
           / \      \
          1   6      14
             / \    /
            4   7  13
```

### The Inorder Sorting Rule

Performing an **Inorder Traversal** (Left $\to$ Root $\to$ Right) on a valid BST visits nodes in **strictly ascending sorted order**:

$$\text{Inorder Output}: [1, 3, 4, 6, 7, 8, 10, 13, 14]$$

---

## 2. Core Operations & Deletion Strategy

| Operation | Average Case (Balanced) | Worst Case (Degenerate Skewed Tree) | Space Complexity |
|---|---|---|---|
| **Search** | **$O(\log N)$** | $O(N)$ | $O(H)$ call stack |
| **Insert** | **$O(\log N)$** | $O(N)$ | $O(H)$ |
| **Delete** | **$O(\log N)$** | $O(N)$ | $O(H)$ |

---

### Deletion in a BST (3 Structural Cases)

```text
Case 1: Deleting a Leaf Node (Node '1')
   Simply remove node 1 and set parent.left = None.

Case 2: Deleting a Node with 1 Child (Node '10')
   Replace node 10 with its single child 14.

Case 3: Deleting a Node with 2 Children (Deleting Root '8')
   1. Find the Inorder Successor (Smallest node in Right Subtree -> Node '10').
   2. Copy Inorder Successor's value into Node '8'.
   3. Recursively delete the Inorder Successor from the right subtree (which falls into Case 1 or 2!).
```

```text
Case 3 Deletion Visual (Delete Node 8):

Original:              Replace 8 with 10 (Inorder Succ):      Delete duplicate 10:
      8                        10                                     10
    /   \                    /   \                                  /   \
   3     12   ──►           3     12             ──►               3     12
        /  \                     /  \                                   /  \
       10  14                   10  14                                 11  14
        \                        \
        11                       11
```

---

## 3. Code Implementations

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

### Operation A: Search & Insertion

```python
def search_bst(root: TreeNode | None, val: int) -> TreeNode | None:
    """
    Search in a BST.
    Time Complexity: O(H)
    Auxiliary Space: O(1) iterative
    """
    curr = root
    while curr:
        if curr.val == val:
            return curr
        elif val < curr.val:
            curr = curr.left
        else:
            curr = curr.right
    return None

def insert_into_bst(root: TreeNode | None, val: int) -> TreeNode:
    """
    Insert value into BST.
    Time Complexity: O(H)
    Auxiliary Space: O(H) recursive
    """
    if not root:
        return TreeNode(val)
        
    if val < root.val:
        root.left = insert_into_bst(root.left, val)
    elif val > root.val:
        root.right = insert_into_bst(root.right, val)
        
    return root
```

---

### Operation B: Deletion in a BST

```python
def delete_node(root: TreeNode | None, key: int) -> TreeNode | None:
    """
    Deletes node with key from BST using Inorder Successor replacement.
    Time Complexity: O(H)
    Auxiliary Space: O(H)
    """
    if not root:
        return None
        
    if key < root.val:
        root.left = delete_node(root.left, key)
    elif key > root.val:
        root.right = delete_node(root.right, key)
    else:
        # Node to delete found!
        
        # Case 1 & 2: 0 or 1 child
        if not root.left:
            return root.right
        elif not root.right:
            return root.left
            
        # Case 3: 2 children -> Find min node in right subtree
        min_node = root.right
        while min_node.left:
            min_node = min_node.left
            
        # Replace root value with inorder successor value
        root.val = min_node.val
        # Delete the duplicate inorder successor node
        root.right = delete_node(root.right, min_node.val)
        
    return root
```

---

## 4. Key Patterns & Algorithms

### Pattern A: Validate Binary Search Tree (Range Bounding DFS)

Given `root`, determine if it is a valid BST.

#### Common Naive Mistake
Checking `node.left.val < node.val < node.right.val` locally for each node is **INSUFFICIENT**! A node deep in the right subtree could be smaller than the root, violating global BST properties.

```text
Invalid Tree local check misses:
        10
       /  \
      5   15
         /  \
        6   20   ◄── '6' is in right subtree of 10, but 6 < 10! INVALID!
```

#### Correct Global Bounding Range Algorithm
Pass valid numeric interval `(low, high)` down the recursion stack:
- Left child range: `(low, node.val)`
- Right child range: `(node.val, high)`

```python
def is_valid_bst(root: TreeNode | None) -> bool:
    """
    Validates BST using Range Bounding DFS.
    Time Complexity: O(N)
    Auxiliary Space: O(H)
    """
    def validate(node: TreeNode | None, low: float, high: float) -> bool:
        if not node:
            return True
            
        if not (low < node.val < high):
            return False
            
        return (validate(node.left, low, node.val) and 
                validate(node.right, node.val, high))

    return validate(root, float('-inf'), float('inf'))
```

---

### Pattern B: Kth Smallest Element in a BST

Find the $K$-th smallest value (1-indexed) in a BST.

#### Optimal Solution: Inorder Traversal Iteration
Since Inorder Traversal visits nodes in ascending sorted order, the $K$-th node visited during Inorder traversal IS the $K$-th smallest element!

```python
def kth_smallest(root: TreeNode | None, k: int) -> int:
    """
    Kth Smallest Element in BST using Iterative Inorder Traversal.
    Time Complexity: O(H + K)
    Auxiliary Space: O(H)
    """
    stack = []
    curr = root
    
    while curr or stack:
        # Push all left nodes
        while curr:
            stack.append(curr)
            curr = curr.left
            
        curr = stack.pop()
        k -= 1
        if k == 0:
            return curr.val
            
        curr = curr.right
        
    return -1
```

---

## 5. Self-Balancing BST Motivation (AVL & Red-Black Trees)

When elements are inserted into a BST in sorted order (`[1, 2, 3, 4, 5]`), the BST degenerates into a linked list of height $O(N)$, causing $O(N)$ time per operation.

Self-balancing BSTs enforce structural invariants using **Tree Rotations** to guarantee $H = O(\log N)$:
- **AVL Trees**: Strict balance factor $|H_L - H_R| \le 1$. Rebalances via Single/Double Rotations.
- **Red-Black Trees**: Relaxes balance using red/black node color properties. Used in C++ `std::map`, Java `TreeMap`, and Linux kernel schedulers.

```text
Left Rotation (Rebalancing Right-Heavy Tree):
    A                   B
     \                 / \
      B      ──►      A   C
       \
        C
```

---

## 6. Practice Problems

### Foundation
- [LeetCode 700: Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/)
- [LeetCode 701: Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/)

### Intermediate
- [LeetCode 98: Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
- [LeetCode 450: Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)
- [LeetCode 230: Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)
- [LeetCode 235: Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

### Advanced
- [LeetCode 99: Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/) ($O(1)$ Space Morris Traversal)
- [LeetCode 108: Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Binary Trees]]
- [[Binary Search]]
- [[Sorting Algorithms]]
- [[DFS]]
