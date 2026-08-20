---
title: Sparse Tables, Range Minimum Queries (RMQ) & Idempotency
aliases:
  - Sparse Table
  - RMQ
  - Range Minimum Query
  - Idempotency
tags:
  - dsa
  - range-query
  - sparse-table
status: complete
---

# Sparse Tables, Range Minimum Queries (RMQ) & Idempotency

## TL;DR

A **Sparse Table** is a static data structure designed for fast range queries. By precomputing answers for all power-of-two intervals ($2^j$) in **$O(N \log N)$ time and space**, a Sparse Table answers **Range Minimum/Maximum/GCD Queries (RMQ)** in **$O(1)$ strict constant time**.

The key requirement for $O(1)$ queries is **Idempotency**: overlapping two range intervals does NOT alter the result.

---

## 1. Idempotency: Why Range Min/Max is $O(1)$ while Sum is $O(\log N)$

An operator $f$ is **Idempotent** if:

$$f(x, x) = x$$

```text
Idempotent Operators (O(1) Query via Overlapping Ranges):
- Min(a, a) = a
- Max(a, a) = a
- GCD(a, a) = a
- Bitwise AND(a, a) = a

Non-Idempotent Operators (Double-Counting Error! Requires O(log N) Non-Overlapping Query):
- Sum(a, a) = 2a  (WRONG if ranges overlap!)
```

---

## 2. Table Precomputation Structure ($O(N \log N)$)

Let `st[i][j]` be the query result for the range starting at index `i` of length $2^j$ (covering interval $[i, \; i + 2^j - 1]$).

```text
Table Index Mapping:
st[i][0] -> Length 2⁰ = 1  (Interval [i, i])
st[i][1] -> Length 2¹ = 2  (Interval [i, i+1])
st[i][2] -> Length 2² = 4  (Interval [i, i+3])
st[i][3] -> Length 2³ = 8  (Interval [i, i+7])
```

### Dynamic Programming Recurrence Equation

To compute `st[i][j]` of length $2^j$, split the interval into two halves of length $2^{j-1}$:
1. First half: `st[i][j-1]` (covers $[i, \; i + 2^{j-1} - 1]$)
2. Second half: `st[i + 2^(j-1)][j-1]` (covers $[i + 2^{j-1}, \; i + 2^j - 1]$)

$$\text{st}[i][j] = \min \left( \text{st}[i][j-1], \;\; \text{st}[i + 2^{j-1}][j-1] \right)$$

```text
Recurrence Combination Visual (j = 3, Length = 8):

Interval [i, i+7]:   ├─────────────────────── Length 8 ───────────────────────┤
First Half (j=2):    ├────────── Length 4 ──────────┤
Second Half (j=2):                                  ├────────── Length 4 ──────────┤
```

---

## 3. $O(1)$ Range Query Mechanics

To answer a query for range $[L, R]$ of length $\text{len} = R - L + 1$:
1. Calculate largest power of 2 that fits inside $\text{len}$:

$$K = \lfloor \log_2(R - L + 1) \rfloor = (R - L + 1).\text{bit\_length}() - 1$$

2. Cover $[L, R]$ using two overlapping intervals of length $2^K$:
   - Block 1 starts at $L$: `st[L][K]` (covers $[L, \; L + 2^K - 1]$)
   - Block 2 ends at $R$: `st[R - 2^K + 1][K]` (covers $[R - 2^K + 1, \; R]$)

$$\text{RangeMin}(L, R) = \min \left( \text{st}[L][K], \;\; \text{st}[R - 2^K + 1][K] \right)$$

```text
O(1) Overlapping Coverage Visual for Range [L, R]:

Query Range [L, R]:       [ L ─────────────────────────────────────────── R ]
Block 1 (starts at L):    [ L ───────────── 2^K ───────────── ]
Block 2 (ends at R):                        [ ───────────── 2^K ───────────── R ]
                          ▲                                                     ▲
                  Overlapping region evaluated twice, but min(x, x) = x!
```

---

## 4. Complete Python Implementation

```python
import math

class SparseTable:
    """
    Sparse Table for static Range Minimum Queries (RMQ).
    Precomputation Time: O(N log N)
    Query Time: O(1)
    Space Complexity: O(N log N)
    """
    def __init__(self, nums: list[int]):
        self.n = len(nums)
        if self.n == 0:
            return
            
        self.k = (self.n).bit_length() # max log2 level
        self.st = [[0] * self.k for _ in range(self.n)]
        
        # Base case j = 0 (intervals of length 2⁰ = 1)
        for i in range(self.n):
            self.st[i][0] = nums[i]
            
        # DP precomputation
        for j in range(1, self.k):
            length = 1 << (j - 1)
            for i in range(self.n - (1 << j) + 1):
                self.st[i][j] = min(self.st[i][j-1], self.st[i + length][j-1])

    def query_min(self, left: int, right: int) -> int:
        """
        Returns minimum value in range [left, right] in O(1) time.
        """
        length = right - left + 1
        k = length.bit_length() - 1 # K = floor(log2(length))
        
        return min(self.st[left][k], self.st[right - (1 << k) + 1][k])
```

---

## 5. Common Pitfalls & Edge Cases

1. **Attempting Point Updates**: Sparse Tables are **STATIC ONLY**. A single element update requires re-running $O(N \log N)$ precomputation! If updates are required, use a [[Segment Tree]] or [[Fenwick Tree]].
2. **Incorrect $K$ Floor Calculation**: Using `int(math.log2(len))` in high-frequency query loops introduces floating-point overhead. Use bitwise `(length).bit_length() - 1` for $O(1)$ integer log calculation.
3. **Out-of-Bounds in Precomputation**: In inner loop `for i in range(...)`, ensure `i + (1 << j) <= N` to prevent index out of bounds.

---

## 6. Practice Problems

### Foundation
- [Spoj RMQSQ: Range Minimum Query](https://www.spoj.com/problems/RMQSQ/)
- [LeetCode 3163: String Compression III](https://leetcode.com/problems/string-compression-iii/)

### Intermediate
- [LeetCode 239: Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) (Static alternative to Monotonic Deque)
- [Lowest Common Ancestor (LCA) via RMQ Reduction](https://cp-algorithms.com/graph/lca_rmq.html) (Euler Tour + RMQ Sparse Table $O(1)$ LCA!)

### Advanced
- [Codeforces 1548B: Integers Have Friends](https://codeforces.com/problemset/problem/1548/B) (Sparse Table for Range GCD + Binary Search)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Segment Tree]]
- [[Fenwick Tree]]
- [[Lowest Common Ancestor]]
- [[Prefix Sum]]
