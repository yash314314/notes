---
title: Trie (Prefix Tree), Autocomplete & Bitwise XOR Trees
aliases:
  - Trie
  - Prefix Tree
  - Radix Tree
  - Binary Trie
tags:
  - dsa
  - trie
  - data-structures
status: complete
---

# Trie (Prefix Tree), Autocomplete & Bitwise XOR Trees

## TL;DR

A **Trie** (pronounced "try", originating from "re**trie**val") is a tree-based search data structure used to store and retrieve associative keys, typically strings. Nodes in a Trie do not store their full key; instead, a node's position in the tree defines its associated key prefix.

Operations (`insert`, `search`, `starts_with`) run in **$O(L)$ time**, where $L$ is the length of the string, completely independent of the number of keys $N$ stored in the Trie.

---

## 1. Node Structure & Visual Anatomy

```text
Words Stored: ["cat", "car", "card", "do", "dog"]

                     (Root)
                    /      \
                  'c'      'd'
                   │        │
                  'a'      'o'*  ──► Word: "do"
                 /   \       │
               't'*  'r'*   'g'* ──► Word: "dog"
                      │
                     'd'*        ──► Word: "card"

'*' Indicates is_end_of_word = True
```

---

## 2. Trie vs Hash Table Comparison

| Operation / Feature | Hash Table (`unordered_set`) | Trie (Prefix Tree) |
|---|---|---|
| **Exact Word Search (`search`)** | $O(L)$ average ($O(N \cdot L)$ worst) | **$O(L)$ deterministic** |
| **Prefix Match (`starts_with`)** | $O(N \cdot L)$ (Iterate all keys) | **$O(L)$** |
| **Find All Words with Prefix** | $O(N \cdot L)$ | **$O(L + K)$** ($K = \text{matches}$) |
| **Sorted Lexicographical Order**| Requires sorting $O(N \log N)$ | **Built-in DFS traversal** |
| **Memory Overhead** | Low | High (Pointers per child character) |

---

## 3. Standard Trie Implementation

```python
class TrieNode:
    def __init__(self):
        # Maps character -> TrieNode
        self.children = {}
        self.is_end_of_word = False

class Trie:
    """
    Standard Prefix Tree (Trie).
    Insert, Search, StartsWith Time Complexity: O(L) where L is word length
    Space Complexity: O(Total characters across all words)
    """
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        """Inserts a word into the Trie."""
        curr = self.root
        for char in word:
            if char not in curr.children:
                curr.children[char] = TrieNode()
            curr = curr.children[char]
        curr.is_end_of_word = True

    def search(self, word: str) -> bool:
        """Returns True if the exact word exists in the Trie."""
        curr = self._navigate(word)
        return curr is not None and curr.is_end_of_word

    def startsWith(self, prefix: str) -> bool:
        """Returns True if any word in the Trie starts with the given prefix."""
        return self._navigate(prefix) is not None

    def _navigate(self, text: str) -> TrieNode | None:
        curr = self.root
        for char in text:
            if char not in curr.children:
                return None
            curr = curr.children[char]
        return curr
```

---

## 4. Advanced Bitwise Trie (Binary Trie for Max XOR)

Given an array of integers `nums`, find the **maximum XOR pair** (`nums[i] ^ nums[j]`).

### Bitwise Choice Invariant
To maximize XOR ($A \oplus B$), at each bit position from bit 31 down to 0, we greedily search for the **opposite bit** ($1 \oplus 0 = 1$):
- If current bit of number is `0`, try to branch to child `1`.
- If current bit of number is `1`, try to branch to child `0`.

```text
Bitwise Trie for 3-bit Integers (5 = '101', 2 = '010'):

                     (Root)
                    /      \
                  '0'      '1'
                   │        │
                  '1'      '0'
                   │        │
                  '0'      '1' (5)
```

#### Python Implementation ($O(32 \cdot N)$ Time)

```python
class BinaryTrieNode:
    def __init__(self):
        self.children = {} # Maps 0 or 1 -> BinaryTrieNode

class MaximumXOR:
    """
    Find Maximum XOR Pair using Binary Trie.
    Time Complexity: O(32 · N) = O(N)
    Auxiliary Space: O(32 · N)
    """
    def __init__(self):
        self.root = BinaryTrieNode()

    def insert(self, num: int):
        curr = self.root
        for i in range(31, -1, -1):
            bit = (num >> i) & 1
            if bit not in curr.children:
                curr.children[bit] = BinaryTrieNode()
            curr = curr.children[bit]

    def get_max_xor(self, num: int) -> int:
        curr = self.root
        max_xor = 0
        for i in range(31, -1, -1):
            bit = (num >> i) & 1
            toggled_bit = 1 - bit # Opposite bit for max XOR!
            
            if toggled_bit in curr.children:
                max_xor |= (1 << i) # Set bit i to 1
                curr = curr.children[toggled_bit]
            else:
                curr = curr.children[bit]
                
        return max_xor

def find_maximum_xor(nums: list[int]) -> int:
    trie = MaximumXOR()
    for num in nums:
        trie.insert(num)
        
    max_result = 0
    for num in nums:
        max_result = max(max_result, trie.get_max_xor(num))
        
    return max_result
```

---

## 5. Common Pitfalls & Edge Cases

1. **Memory Allocation Over-head for Fixed Arrays**: Using `self.children = [None] * 26` for every node creates 26 null pointers per node, consuming massive memory. Using a Python dictionary `self.children = {}` allocates memory on-demand.
2. **Confusing Prefix Match with Word Match**: In `search("cat")`, matching characters `'c'`, `'a'`, `'t'` is insufficient if `is_end_of_word` is `False` (e.g. only `"caterpillar"` was inserted).
3. **Word Deletion Reference Leak**: Deleting a word without un-linking unused leaf nodes leaves orphan dead branches in memory.

---

## 6. Practice Problems

### Foundation
- [LeetCode 208: Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/)
- [LeetCode 14: Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

### Intermediate
- [LeetCode 211: Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) (Wildcard '.' Matching)
- [LeetCode 677: Map Sum Pairs](https://leetcode.com/problems/map-sum-pairs/)
- [LeetCode 421: Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

### Advanced
- [LeetCode 212: Word Search II](https://leetcode.com/problems/word-search-ii/) (Grid Backtracking + Trie Pruning)
- [LeetCode 1707: Maximum XOR With an Element From Array](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/) (Offline Queries + Binary Trie)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Strings]]
- [[Trees]]
- [[DFS]]
- [[Bit Manipulation]]
