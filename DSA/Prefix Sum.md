---
title: Prefix Sum & Difference Array Techniques
aliases:
  - Prefix Sum
  - Prefix Array
  - 2D Prefix Sum
  - Difference Array
tags:
  - dsa
  - patterns
  - prefix-sum
status: complete
---

# Prefix Sum & Difference Array Techniques

## TL;DR

**Prefix Sum** is a preprocessing technique that replaces repeated range summation queries ($O(N)$ per query) with constant-time lookup ($O(1)$ per query) after a single $O(N)$ precomputation pass.

By storing cumulative aggregates in an auxiliary array, any range sum query $[L, R]$ is computed as:

$$\text{RangeSum}(L, R) = \text{Prefix}[R + 1] - \text{Prefix}[L]$$

---

## 1. 1D Prefix Sum Construction & Intuition

Given an array $A$ of size $N$, construct a Prefix Array $P$ of size $N + 1$ initialized with $P[0] = 0$.

$$P[i] = \sum_{k=0}^{i-1} A[k] \quad \implies \quad P[i] = P[i-1] + A[i-1]$$

### ASCII Memory Layout & 1-Based Offset Trick

```text
Original Array A (Size 5):
Index:      0     1     2     3     4
         ┌─────┬─────┬─────┬─────┬─────┐
Values:  │  3  │  1  │  4  │  1  │  5  │
         └─────┴─────┴─────┴─────┴─────┘

Prefix Array P (Size 6, 1-based shift):
Index:      0     1     2     3     4     5
         ┌─────┬─────┬─────┬─────┬─────┬─────┐
Values:  │  0  │  3  │  4  │  8  │  9  │ 14  │
         └─────┴─────┴─────┴─────┴─────┴─────┘

Query: RangeSum(L=1, R=3)  (Elements A[1] + A[2] + A[3] = 1 + 4 + 1 = 6)
Calculation: P[R + 1] - P[L] = P[4] - P[1] = 9 - 3 = 6  (O(1) Constant Time!)
```

---

## 2. 2D Prefix Sum (Matrix Subgrid Range Queries)

For a 2D matrix of dimensions $M \times N$, we precompute 2D Prefix Matrix $P$ of size $(M+1) \times (N+1)$.

### Precomputation Formula (Inclusion-Exclusion)

$$P[i][j] = P[i-1][j] + P[i][j-1] - P[i-1][j-1] + \text{Matrix}[i-1][j-1]$$

```text
2D Precomputation Inclusion-Exclusion:

┌─────────────────┬──────────┐
│                 │          │
│   P[i-1][j-1]   │          │  P[i-1][j] (Top Area)
│    (Overlapped) │          │
├─────────────────┼──────────┤
│                 │  Cell    │
│  P[i][j-1]      │ (i-1,j-1)│  P[i][j] (Total Combined)
└─────────────────┴──────────┘
```

### Range Query Formula ($O(1)$ Subgrid Sum)

Sum of subgrid from top-left $(r_1, c_1)$ to bottom-right $(r_2, c_2)$:

$$\text{SubgridSum} = P[r_2+1][c_2+1] - P[r_1][c_2+1] - P[r_2+1][c_1] + P[r_1][c_1]$$

#### Python Implementation

```python
class NumMatrix:
    """
    2D Immutable Matrix Range Sum Query.
    Precomputation: O(M · N)
    Query: O(1)
    Space: O(M · N)
    """
    def __init__(self, matrix: list[list[int]]):
        if not matrix or not matrix[0]:
            return
        m, n = len(matrix), len(matrix[0])
        self.P = [[0] * (n + 1) for _ in range(m + 1)]
        
        for r in range(1, m + 1):
            for c in range(1, n + 1):
                self.P[r][c] = (self.P[r-1][c] + 
                                self.P[r][c-1] - 
                                self.P[r-1][c-1] + 
                                matrix[r-1][c-1])

    def sum_region(self, r1: int, c1: int, r2: int, c2: int) -> int:
        return (self.P[r2 + 1][c2 + 1] - 
                self.P[r1][c2 + 1] - 
                self.P[r2 + 1][c1] + 
                self.P[r1][c1])
```

---

## 3. Difference Array (O(1) Range Updates)

When multiple range update operations are performed on an initial array of zeros (e.g., "Add $V$ to all elements in range $[L, R]$"), applying updates directly takes $O(N)$ per operation.

A **Difference Array** $D$ optimizes range updates to **$O(1)$**:

$$D[i] = A[i] - A[i-1]$$

### Range Update Rule $[L, R, +V]$

```text
To add V to range [L, R]:
1. D[L]   += V  (Starts adding V from index L onwards)
2. D[R+1] -= V  (Cancels out addition of V from index R+1 onwards)
```

```text
Difference Array Snapshot for Update Range [1, 3, +5]:

Index:      0     1     2     3     4     5
Diff D:   [ 0  , +5  ,  0  ,  0  , -5  ,  0 ]
                                   ▲
                                   R+1 cancels out +5

Reconstruct final values via Prefix Sum of D:
Prefix:   [ 0  ,  5  ,  5  ,  5  ,  0  ,  0 ]
Range:           ├─── Elements 1..3 boosted by 5 ───┤
```

---

## 4. Key Pattern: Subarray Sum Equals K ($O(N)$ Prefix + Hash Map)

Given an array of integers `nums` and an integer `k`, return the total number of subarrays whose sum equals `k`.

### Insight Shift

Let $P[j]$ be the prefix sum up to index $j$.
A contiguous subarray from $i$ to $j$ has sum:

$$\text{SubarraySum}(i, j) = P[j+1] - P[i] = k \quad \implies \quad P[i] = P[j+1] - k$$

Instead of testing all pairs $(i, j)$ in $O(N^2)$, we store previous prefix sum frequencies in a Hash Map. For current prefix sum $P$, we add `map[P - k]` to our total count!

#### Python Implementation

```python
def subarray_sum(nums: list[int], k: int) -> int:
    """
    Subarray Sum Equals K using Prefix Sum + Hash Map.
    Time Complexity: O(N)
    Auxiliary Space: O(N)
    """
    prefix_counts = {0: 1} # Base case: prefix sum of 0 seen once
    current_prefix_sum = 0
    total_count = 0
    
    for num in nums:
        current_prefix_sum += num
        needed_prefix = current_prefix_sum - k
        
        if needed_prefix in prefix_counts:
            total_count += prefix_counts[needed_prefix]
            
        prefix_counts[current_prefix_sum] = prefix_counts.get(current_prefix_sum, 0) + 1
        
    return total_count
```

---

## 5. Practice Problems

### Foundation
- [LeetCode 303: Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)
- [LeetCode 724: Find Pivot Index](https://leetcode.com/problems/find-pivot-index/)

### Intermediate
- [LeetCode 560: Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)
- [LeetCode 304: Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/)
- [LeetCode 1109: Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) (Difference Array)

### Advanced
- [LeetCode 523: Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/) (Prefix Modulo)
- [LeetCode 974: Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Arrays]]
- [[Hashing]]
- [[Sliding Window]]
- [[Fenwick Tree]]
