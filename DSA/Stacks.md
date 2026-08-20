---
title: Stacks, Monotonic Stacks & Expression Processing
aliases:
  - Stacks
  - Stack
  - Monotonic Stack
  - Min Stack
tags:
  - dsa
  - linear-data-structures
  - stacks
status: complete
---

# Stacks, Monotonic Stacks & Expression Processing

## TL;DR

A **Stack** is an abstract linear data structure operating under the **LIFO (Last-In, First-Out)** principle: the item inserted most recently is the first to be removed. All core operations (`push`, `pop`, `peek`) execute in strict **$O(1)$ constant time**. Stacks are fundamental to function call management, expression parsing, depth-first search (DFS), and monotonic range queries.

---

## 1. What is a Stack? Real-World Analogy & Memory Layout

### Real-World Analogy: Stack of Cafeteria Trays

```text
        ┌───────────┐
        │  Tray 3   │ ◄── Top of Stack (Pushed last, Popped first)
        ├───────────┤
        │  Tray 2   │
        ├───────────┤
        │  Tray 1   │ ◄── Bottom of Stack (Pushed first)
        └───────────┘

Operation Sequence: Push(1) ──► Push(2) ──► Push(3) ──► Pop() [Returns 3]
```

### Relationship to System Memory: The CPU Call Stack

Every function invocation creates a **Stack Frame** stored in the CPU Call Stack. When `functionA()` calls `functionB()`, `functionB`'s frame is pushed onto the stack. When `functionB()` finishes, its frame is popped, returning execution to `functionA()`.

```text
High Memory Address
┌─────────────────────────┐
│ main() Frame            │
├─────────────────────────┤
│ processData() Frame     │
├─────────────────────────┤
│ computeSum() Frame      │ ◄── Current Active Stack Pointer (ESP/RSP)
└─────────────────────────┘
Low Memory Address
```

---

## 2. Core Operations & Performance Guarantees

| Operation | Description | Time Complexity | Auxiliary Space |
|---|---|---|---|
| **`push(x)`** | Insert element `x` onto the top of the stack | **$O(1)$** | $O(1)$ |
| **`pop()`** | Remove and return the top element from the stack | **$O(1)$** | $O(1)$ |
| **`peek()` / `top()`** | Read the top element without removing it | **$O(1)$** | $O(1)$ |
| **`is_empty()`** | Return `True` if stack contains no elements | **$O(1)$** | $O(1)$ |
| **`size()`** | Return total count of elements | **$O(1)$** | $O(1)$ |

---

## 3. Implementation Strategies: Dynamic Array vs Linked List

### 1. Dynamic Array Stack (Recommended)
Uses `std::vector` (C++) or standard `list` (Python). Elements are stored in contiguous memory, maximizing CPU cache locality. Resizing happens amortized $O(1)$.

### 2. Linked List Stack
Uses a singly linked list where `push` inserts at `head` and `pop` removes from `head`. Avoids resizing spikes but incurs $O(N)$ pointer allocation overhead.

```python
class ArrayStack:
    def __init__(self):
        self._data = []
        
    def push(self, val: int) -> None:
        self._data.append(val)
        
    def pop(self) -> int:
        if self.is_empty():
            raise IndexError("pop from empty stack")
        return self._data.pop()
        
    def peek(self) -> int:
        if self.is_empty():
            raise IndexError("peek from empty stack")
        return self._data[-1]
        
    def is_empty(self) -> bool:
        return len(self._data) == 0
```

---

## 4. Key Patterns & Algorithms

### Pattern A: Min Stack ($O(1)$ Minimum Element Access)

Design a stack that supports `push`, `pop`, `top`, and retrieving the minimum element (`getMin`) in $O(1)$ time.

#### Intuition
Maintain a parallel **Auxiliary Stack** that stores the minimum element seen so far at each corresponding stack level.

```text
Action       Primary Stack      Auxiliary Min Stack
---------------------------------------------------
Push(5)      [5]                [5]
Push(3)      [5, 3]             [5, 3]           (min(3, 5) = 3)
Push(7)      [5, 3, 7]          [5, 3, 3]        (min(7, 3) = 3)
Push(2)      [5, 3, 7, 2]       [5, 3, 3, 2]     (min(2, 3) = 2)
getMin() -> Returns 2 (Top of Min Stack)
Pop()        [5, 3, 7]          [5, 3, 3]
getMin() -> Returns 3 (Top of Min Stack)
```

#### Python Implementation

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)
        current_min = val if not self.min_stack else min(val, self.min_stack[-1])
        self.min_stack.append(current_min)

    def pop(self) -> None:
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

---

### Pattern B: Monotonic Stack (Next Greater Element $O(N)$)

A **Monotonic Stack** maintains its elements in strictly increasing or decreasing order.

- **Monotonic Decreasing Stack**: Used to find the **Next Greater Element**. Elements are evicted when a larger incoming element arrives.
- **Monotonic Increasing Stack**: Used to find the **Next Smaller Element**.

#### Problem: Next Greater Element I

Given an array `nums`, find the next strictly greater element for each index. If no greater element exists, output `-1`.

#### ASCII State Trace for `nums = [2, 1, 5, 6, 2, 3]`

```text
Index i=0 (val 2): Stack = [0 (val 2)]
Index i=1 (val 1): 1 < 2 -> Stack = [0 (val 2), 1 (val 1)]
Index i=2 (val 5): 
  - 5 > 1 -> Pop index 1 (val 1). Next Greater for 1 is 5!
  - 5 > 2 -> Pop index 0 (val 2). Next Greater for 2 is 5!
  - Push index 2 -> Stack = [2 (val 5)]
Index i=3 (val 6):
  - 6 > 5 -> Pop index 2 (val 5). Next Greater for 5 is 6!
  - Push index 3 -> Stack = [3 (val 6)]
Index i=4 (val 2): 2 < 6 -> Stack = [3 (val 6), 4 (val 2)]
Index i=5 (val 3):
  - 3 > 2 -> Pop index 4 (val 2). Next Greater for 2 is 3!
  - Push index 5 -> Stack = [3 (val 6), 5 (val 3)]

Remaining indices in stack [3, 5] have no next greater -> result = -1.
```

#### Amortized Analysis Proof

Even though there is a `while` loop inside the iteration:
- Each index is **pushed onto the stack exactly once**.
- Each index is **popped from the stack at most once**.

$$\text{Total Stack Operations} \le 2N \implies \text{Amortized Time Complexity}: O(N)$$

#### Python Implementation

```python
def next_greater_element(nums: list[int]) -> list[int]:
    """
    Next Greater Element using Monotonic Decreasing Stack.
    Time Complexity: O(N) amortized
    Auxiliary Space: O(N)
    """
    n = len(nums)
    result = [-1] * n
    stack = [] # Stores indices
    
    for i in range(n):
        # Evict indices whose value is smaller than current element nums[i]
        while stack and nums[stack[-1]] < nums[i]:
            popped_idx = stack.pop()
            result[popped_idx] = nums[i]
            
        stack.append(i)
        
    return result
```

---

## 5. Common Pitfalls & Edge Cases

1. **Stack Underflow Error**: Calling `pop()` or `top()` on an empty stack raises exceptions. Always verify `if stack:` before popping.
2. **Missing Opening Parentheses**: When processing expressions (e.g., `Valid Parentheses`), receiving a closing bracket `')'` when the stack is empty indicates an invalid expression.
3. **Monotonic Stack Duplicate Values**: Clarify whether "Next Greater" requires strictly greater (`>`) or greater-or-equal (`>=`).

---

## 6. Practice Problems

### Foundation
- [LeetCode 20: Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)
- [LeetCode 155: Min Stack](https://leetcode.com/problems/min-stack/)

### Intermediate
- [LeetCode 739: Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) (Monotonic Stack)
- [LeetCode 503: Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) (Circular Array)
- [LeetCode 150: Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

### Advanced
- [LeetCode 84: Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)
- [LeetCode 85: Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Arrays]]
- [[Linked Lists]]
- [[Queues]]
- [[DFS]]
- [[Monotonic Stack]]
