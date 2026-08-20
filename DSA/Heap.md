---
title: Heaps, Priority Queues & Linear Build-Heap
aliases:
  - Heap
  - Min Heap
  - Max Heap
  - Priority Queue
  - Heapify
tags:
  - dsa
  - heap
  - data-structures
status: complete
---

# Heaps, Priority Queues & Linear Build-Heap

## TL;DR

A **Binary Heap** is a specialized **Complete Binary Tree** stored inside a 1D contiguous array. It satisfies the **Heap Order Invariant**:
- **Min-Heap**: The value of every parent node is $\le$ the values of its children. Root is the global minimum.
- **Max-Heap**: The value of every parent node is $\ge$ the values of its children. Root is the global maximum.

A Heap provides **$O(1)$ constant time access to the min/max element** and **$O(\log N)$ insertion and deletion**.

---

## 1. Array Memory Representation of Binary Heap

Because a heap is a **Complete Binary Tree** (all levels filled except possibly the last, which is filled left-to-right), it requires zero pointers! It maps directly into a 1D Array using index arithmetic:

```text
Tree Representation:
                 10 (Index 0)
               /    \
         15 (Idx 1)   20 (Idx 2)
        /    \
  30 (Idx 3) 40 (Idx 4)

Array Representation:
Index:   0    1    2    3    4
       ┌────┬────┬────┬────┬────┐
Value: │ 10 │ 15 │ 20 │ 30 │ 40 │
       └────┴────┴────┴────┴────┘
```

### Zero-Indexed Formula Matrix

$$\text{Left Child}(i) = 2i + 1$$

$$\text{Right Child}(i) = 2i + 2$$

$$\text{Parent}(i) = \left\lfloor \frac{i - 1}{2} \right\rfloor$$

---

## 2. Operations & Mathematical Complexity Analysis

| Operation | Description | Time Complexity | Auxiliary Space |
|---|---|---|---|
| **`peek_min()`** | Access root element | **$O(1)$** | $O(1)$ |
| **`insert(x)`** | Append at end + `sift_up` | **$O(\log N)$** | $O(1)$ |
| **`extract_min()`** | Swap root with last + `sift_down` | **$O(\log N)$** | $O(1)$ |
| **`heapify(arr)`** | Build Heap from unsorted array | **$O(N)$** (Linear!) | $O(1)$ in-place |

---

## 3. Mathematical Proof: Why `build_heap` is $O(N)$, Not $O(N \log N)$

Naive insertion of $N$ elements one by one takes $N \times O(\log N) = O(N \log N)$ time.
However, **bottom-up `heapify`** starts from the last non-leaf node and calls `sift_down` downwards.

```text
Tree Nodes per Height Level:
Height h=0 (Leaves): N/2 nodes -> 0 sift down steps
Height h=1:          N/4 nodes -> 1 sift down step
Height h=2:          N/8 nodes -> 2 sift down steps
...
Height h:            N / 2^(h+1) nodes -> h steps
```

### Sum of Series Derivation

$$T(N) = \sum_{h=0}^{\log N} \frac{N}{2^{h+1}} \cdot h = \frac{N}{2} \sum_{h=0}^{\infty} \frac{h}{2^h}$$

Let $S = \sum_{h=0}^{\infty} \frac{h}{2^h} = \frac{0}{1} + \frac{1}{2} + \frac{2}{4} + \frac{3}{8} + \dots = 2$.

$$T(N) = \frac{N}{2} \times 2 = \Theta(N)$$

$$\text{Building a heap bottom-up takes strictly } \Theta(N) \text{ Linear Time!}$$

---

## 4. Fundamental Operations: Sift-Up & Sift-Down

```text
Sift-Up (Bubble Up on Insert):
  Append element at array end.
  While element < parent:
      Swap(element, parent)
      i = parent(i)

Sift-Down (Sink Down on Extract Min):
  Swap root with last element, then pop last element.
  While element > min(left_child, right_child):
      Swap(element, smaller_child)
      i = smaller_child_index
```

### Complete Python Min-Heap Implementation

```python
class MinHeap:
    def __init__(self):
        self.heap = []

    def peek(self) -> int:
        if not self.heap:
            raise IndexError("peek from empty heap")
        return self.heap[0]

    def push(self, val: int) -> None:
        self.heap.append(val)
        self._sift_up(len(self.heap) - 1)

    def pop(self) -> int:
        if not self.heap:
            raise IndexError("pop from empty heap")
        
        root_val = self.heap[0]
        last_val = self.heap.pop()
        
        if self.heap:
            self.heap[0] = last_val
            self._sift_down(0)
            
        return root_val

    def _sift_up(self, i: int) -> None:
        parent = (i - 1) // 2
        while i > 0 and self.heap[i] < self.heap[parent]:
            self.heap[i], self.heap[parent] = self.heap[parent], self.heap[i]
            i = parent
            parent = (i - 1) // 2

    def _sift_down(self, i: int) -> None:
        n = len(self.heap)
        while True:
            smallest = i
            left = 2 * i + 1
            right = 2 * i + 2
            
            if left < n and self.heap[left] < self.heap[smallest]:
                smallest = left
            if right < n and self.heap[right] < self.heap[smallest]:
                smallest = right
                
            if smallest != i:
                self.heap[i], self.heap[smallest] = self.heap[smallest], self.heap[i]
                i = smallest
            else:
                break
```

---

## 5. Core Priority Queue Algorithmic Patterns

### Pattern A: Top K Elements ($O(N \log K)$ Time)

Find the $K$ largest elements in an unsorted array.

#### Min-Heap Optimization Strategy
Maintain a **Min-Heap of size $K$**:
- Iterate through elements. Push each element into the Min-Heap.
- If heap size exceeds $K$, pop the root (which evicts the smallest element).
- After $N$ iterations, the Min-Heap contains the **$K$ largest elements**!

$$\text{Time Complexity}: O(N \log K) \ll O(N \log N) \text{ (Sorting)}$$

```python
import heapq

def find_kth_largest(nums: list[int], k: int) -> int:
    """
    Find Kth Largest Element using Min-Heap of size K.
    Time Complexity: O(N log K)
    Auxiliary Space: O(K)
    """
    min_heap = []
    
    for num in nums:
        heapq.heappush(min_heap, num)
        if len(min_heap) > k:
            heapq.heappop(min_heap)
            
    return min_heap[0]
```

---

### Pattern B: Find Median from Data Stream (Two Heaps Pattern)

Maintain the median of a continuous stream of incoming numbers in $O(\log N)$ insert, $O(1)$ find.

#### Two-Heap Strategy
- **Max-Heap (`lower_half`)**: Stores the smaller half of numbers (root is max of lower half).
- **Min-Heap (`upper_half`)**: Stores the larger half of numbers (root is min of upper half).

```text
Stream: [5, 2, 8, 1, 9]

Max-Heap (Lower Half)     Min-Heap (Upper Half)
       [2, 1]                    [8, 9]
      max = 2                   min = 8

Median = (2 + 8) / 2.0 = 5.0 (If total items even)
Median = max(lower_half)     (If lower_half has 1 extra item)
```

```python
class MedianFinder:
    """
    Find Median from Data Stream using Two Heaps.
    Insert Time: O(log N)
    Find Median Time: O(1)
    Space: O(N)
    """
    def __init__(self):
        self.small = [] # Max-heap (invert numbers: -x)
        self.large = [] # Min-heap

    def addNum(self, num: int) -> None:
        # Push to max-heap small
        heapq.heappush(self.small, -num)
        
        # Ensure max of small <= min of large
        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
            
        # Balance sizes (small can have at most 1 extra item)
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        elif len(self.large) > len(self.small):
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)

    def findMedian(self) -> float:
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2.0
```

---

## 6. Common Pitfalls & Edge Cases

1. **Python `heapq` Max-Heap Trap**: `heapq` in Python is strictly a **Min-Heap**. To implement a Max-Heap, invert values on push (`-val`) and invert back on pop (`-heappop()`).
2. **Mutating Heap Elements in-place**: Modifying an element inside a heap array breaks the heap invariant without triggering `sift_up` or `sift_down`.
3. **Empty Heap Pops**: Popping from an empty heap raises an index error. Always check `if heap:` first.

---

## 7. Practice Problems

### Foundation
- [LeetCode 703: Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)
- [LeetCode 1046: Last Stone Weight](https://leetcode.com/problems/last-stone-weight/)

### Intermediate
- [LeetCode 215: Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
- [LeetCode 347: Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
- [LeetCode 973: K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

### Advanced
- [LeetCode 295: Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)
- [LeetCode 23: Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Arrays]]
- [[Binary Trees]]
- [[Sorting Algorithms]]
- [[Greedy Algorithms]]
