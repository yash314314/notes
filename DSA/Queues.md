---
title: Queues, Circular Queues & Monotonic Deques
aliases:
  - Queues
  - Queue
  - Circular Queue
  - Deque
  - Monotonic Deque
tags:
  - dsa
  - linear-data-structures
  - queues
status: complete
---

# Queues, Circular Queues & Monotonic Deques

## TL;DR

A **Queue** is a linear data structure operating under the **FIFO (First-In, First-Out)** principle: the item inserted first is the first to be removed. All fundamental operations (`enqueue`, `dequeue`, `front`) execute in **$O(1)$ constant time**. Queues are indispensable for Breadth-First Search (BFS), asynchronous job processing, CPU scheduling queues, and sliding-window sliding-max optimizations using **Deques**.

---

## 1. What is a Queue? Real-World Analogy & Hardware Context

### Real-World Analogy: Ticket Counter Line

```text
EXIT ◄── [ Person A ] ◄── [ Person B ] ◄── [ Person C ] ◄── ENTER
            (Front)                           (Rear / Tail)

Person A leaves first (Dequeue). Person D joins at the back (Enqueue).
```

### System Architecture Context
- **OS Task Scheduler**: Ready processes wait in a FIFO queue for CPU time slices.
- **Network Interface Card (NIC)**: Incoming packets buffer in a ring queue before the kernel reads them.

---

## 2. Core Queue Operations & Performance

| Operation | Description | Time Complexity | Auxiliary Space |
|---|---|---|---|
| **`enqueue(x)` / `push(x)`** | Insert element `x` at the rear/tail of the queue | **$O(1)$** | $O(1)$ |
| **`dequeue()` / `pop()`** | Remove and return element from the front/head | **$O(1)$** | $O(1)$ |
| **`front()` / `peek()`** | Access element at the front without removing | **$O(1)$** | $O(1)$ |
| **`is_empty()`** | Check if queue contains zero elements | **$O(1)$** | $O(1)$ |

---

## 3. Implementations: Circular Array vs Linked List

### Why Plain Arrays Fail for Queues
If a queue uses a standard array and `dequeue` removes element at index 0, every remaining element must shift left by 1 position, turning `dequeue` into a slow **$O(N)$** operation.

---

### Strategy A: Circular Array Queue ($O(1)$ Modulo Arithmetic)

A **Circular Queue** fixes the $O(N)$ shift penalty by wrapping `head` and `tail` pointers around the array using modulo arithmetic:

$$\text{Next Index} = (\text{Current Index} + 1) \pmod{\text{Capacity}}$$

```text
Array Capacity = 5:

Initial (Enqueue 10, 20, 30):
Index:     0    1    2    3    4
        ┌────┬────┬────┬────┬────┐
Values: │ 10 │ 20 │ 30 │ _  │ _  │
        └────┴────┴────┴────┴────┘
          ▲         ▲
        head       tail

After 2 Dequeues (Head moves right):
Index:     0    1    2    3    4
        ┌────┬────┬────┬────┬────┐
Values: │ _  │ _  │ 30 │ _  │ _  │
        └────┴────┴────┴────┴────┘
                    ▲
                head/tail

After Enqueue 40, 50, 60 (Tail wraps around to Index 0!):
Index:     0    1    2    3    4
        ┌────┬────┬────┬────┬────┐
Values: │ 60 │ _  │ 30 │ 40 │ 50 │
        └────┴────┴────┴────┴────┘
          ▲         ▲
        tail       head
```

#### Python Circular Queue Implementation

```python
class MyCircularQueue:
    """
    Fixed-Capacity Circular Queue using Array + Modulo Pointers.
    Time Complexity: O(1) for all operations
    Auxiliary Space: O(K) where K is capacity
    """
    def __init__(self, k: int):
        self.capacity = k
        self.queue = [0] * k
        self.head = 0
        self.tail = 0
        self.count = 0

    def enQueue(self, value: int) -> bool:
        if self.isFull():
            return False
        self.queue[self.tail] = value
        self.tail = (self.tail + 1) % self.capacity
        self.count += 1
        return True

    def deQueue(self) -> bool:
        if self.isEmpty():
            return False
        self.head = (self.head + 1) % self.capacity
        self.count -= 1
        return True

    def Front(self) -> int:
        return -1 if self.isEmpty() else self.queue[self.head]

    def Rear(self) -> int:
        if self.isEmpty():
            return -1
        # Tail points to next insertion slot, so rear is (tail - 1) % capacity
        return self.queue[(self.tail - 1 + self.capacity) % self.capacity]

    def isEmpty(self) -> bool:
        return self.count == 0

    def isFull(self) -> bool:
        return self.count == self.capacity
```

---

## 4. Deques (Double-Ended Queues)

A **Deque** allows insertions and deletions at **both ends** (`front` and `back`) in $O(1)$ time.

```text
              FRONT                                 REAR
      PushFront ◄───┐                             ┌───► PushBack
                    │ ┌───┬───┬───┬───┬───┬───┐  │
      PopFront  ───►  │ A │ B │ C │ D │ E │ F │  ◄─── PopBack
                      └───┴───┴───┴───┴───┴───┘
```

In Python, `collections.deque` is implemented as a doubly linked block list, guaranteeing $O(1)$ operations on both ends.

---

## 5. Advanced Pattern: Monotonic Deque (Sliding Window Maximum $O(N)$)

Given an array `nums` and a sliding window of size $K$, return the maximum value in each window position as it slides from left to right.

### Key Monotonic Insight

Inside a window, if an incoming element `nums[i]` is **larger than or equal to** existing elements, those smaller elements can **never be the maximum** in the current or any future window!

Therefore, we maintain a **Monotonic Decreasing Deque** of array indices:
1. **Evict smaller elements from the back** of the deque (`while deque and nums[deque[-1]] <= nums[i]`).
2. **Push current index** onto the back.
3. **Evict expired indices from the front** if they fall outside the current window (`if deque[0] <= i - K`).
4. **Front of deque always holds the index of the maximum element** in the current window!

```text
Input: nums = [1, 3, -1, -3, 5, 3, 6, 7], K = 3

Window 1 [1, 3, -1]:
- Push 1 (idx 0): Deque = [0 (val 1)]
- Push 3 (idx 1): 3 > 1 -> Pop back 0. Deque = [1 (val 3)]
- Push -1 (idx 2): -1 < 3 -> Deque = [1 (val 3), 2 (val -1)]
- Max for Window 1 = nums[deque[0]] = nums[1] = 3

Window 2 [3, -1, -3]:
- Push -3 (idx 3): Deque = [1 (val 3), 2 (val -1), 3 (val -3)]
- Max for Window 2 = 3

Window 3 [-1, -3, 5]:
- Push 5 (idx 4): 5 > -3, 5 > -1, 5 > 3 -> Pop ALL back elements!
  Deque = [4 (val 5)]
- Max for Window 3 = 5
```

#### Python Implementation

```python
from collections import deque

def max_sliding_window(nums: list[int], k: int) -> list[int]:
    """
    Sliding Window Maximum using Monotonic Decreasing Deque.
    Time Complexity: O(N) amortized
    Auxiliary Space: O(K) for deque
    """
    q = deque() # Stores indices
    result = []
    
    for i, num in enumerate(nums):
        # 1. Remove indices outside the current sliding window
        if q and q[0] <= i - k:
            q.popleft()
            
        # 2. Maintain monotonic decreasing order (evict smaller values from back)
        while q and nums[q[-1]] <= num:
            q.pop()
            
        # 3. Add current element index
        q.append(i)
        
        # 4. Record max once window reaches size K
        if i >= k - 1:
            result.append(nums[q[0]])
            
    return result
```

---

## 6. Common Pitfalls & Edge Cases

1. **`deque.pop(0)` in Python Lists**: Calling `pop(0)` on a standard Python list takes $O(N)$ time. Always use `collections.deque` and `popleft()` for $O(1)$ operations.
2. **Circular Queue Index Wrap Around**: In circular arrays, forgetting to use `(index + 1) % Capacity` leads to array bounds overflow.
3. **Deque Index Storage**: Storing array values instead of array indices in a monotonic deque makes it impossible to check if the front element has expired out of the window.

---

## 7. Practice Problems

### Foundation
- [LeetCode 225: Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/)
- [LeetCode 232: Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)

### Intermediate
- [LeetCode 622: Design Circular Queue](https://leetcode.com/problems/design-circular-queue/)
- [LeetCode 950: Reveal Cards In Increasing Order](https://leetcode.com/problems/reveal-cards-in-increasing-order/)

### Advanced
- [LeetCode 239: Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) (Monotonic Deque)
- [LeetCode 862: Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/) (Monotonic Deque + Prefix Sum)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Arrays]]
- [[Linked Lists]]
- [[Stacks]]
- [[BFS]]
- [[Sliding Window]]
