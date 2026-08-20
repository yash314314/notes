---
title: Longest Increasing Subsequence (LIS), Patience Sorting & O(N log N) Binary Search
aliases:
  - Longest Increasing Subsequence
  - LIS
  - Patience Sorting
  - Russian Doll Envelopes
tags:
  - dsa
  - dynamic-programming
  - lis
status: complete
---

# Longest Increasing Subsequence (LIS), Patience Sorting & O(N log N) Binary Search

## TL;DR

Given an integer array `nums`, the **Longest Increasing Subsequence (LIS)** problem finds the length of the longest subsequence where all elements are strictly increasing.

While standard dynamic programming yields an **$O(N^2)$** algorithm, applying **Patience Sorting with Binary Search** optimizes the algorithm to **$O(N \log N)$ time and $O(N)$ space**.

---

## 1. Approach 1: $O(N^2)$ Standard Dynamic Programming

### State Definition & Recurrence Relation
Let `dp[i]` be the length of the LIS **ending at index `i`**:

$$\text{dp}[i] = 1 + \max \left( \{ \text{dp}[j] \mid 0 \le j < i \text{ and } \text{nums}[j] < \text{nums}[i] \} \cup \{0\} \right)$$

$$\text{LIS Length} = \max_{0 \le i < N} (\text{dp}[i])$$

#### Python Implementation ($O(N^2)$ Time, $O(N)$ Space)

```python
def length_of_lis_dp(nums: list[int]) -> int:
    """
    Quadratic DP for LIS.
    Time Complexity: O(N²)
    Auxiliary Space: O(N)
    """
    if not nums:
        return 0
        
    n = len(nums)
    dp = [1] * n
    
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], 1 + dp[j])
                
    return max(dp)
```

---

## 2. Approach 2: $O(N \log N)$ Patience Sorting + Binary Search

### The Patience Sorting Principle

Imagine dealing cards into piles on a table following card game rules:
1. A card can be placed on top of an existing pile if it is **$\le$ the top card** of that pile.
2. Otherwise, start a **new pile to the right**.
3. Always pick the **leftmost valid pile** (via Binary Search!).

```text
Patience Sorting Trace for nums = [10, 9, 2, 5, 3, 7, 101, 18]:

1. Process 10  -> Pile 1: [10]                   Tails = [10]
2. Process 9   -> Replace Pile 1: [9]            Tails = [9]
3. Process 2   -> Replace Pile 1: [2]            Tails = [2]
4. Process 5   -> Start Pile 2: [5]              Tails = [2, 5]
5. Process 3   -> Replace Pile 2: [3]            Tails = [2, 3]
6. Process 7   -> Start Pile 3: [7]              Tails = [2, 3, 7]
7. Process 101 -> Start Pile 4: [101]            Tails = [2, 3, 7, 101]
8. Process 18  -> Replace Pile 4: [18]           Tails = [2, 3, 7, 18]

Total Piles Created = 4  ===> LIS Length = 4!
```

---

### The Dynamic `tails` Invariant

Let `tails[k]` store the **smallest tail value** among all increasing subsequences of length `k + 1` found so far.

1. `tails` array is **guaranteed to be strictly sorted** at all times!
2. For each number `x` in `nums`:
   - Use **Binary Search (`bisect_left`)** to find the index `i` where `x` belongs in `tails`.
   - If `i == len(tails)`, `x` is greater than all tails $\to$ append `x` to `tails` (expand LIS length!).
   - Otherwise, update `tails[i] = x` (greedily lower the smallest tail for length `i + 1`).

#### Python Implementation ($O(N \log N)$ Time, $O(N)$ Space)

```python
import bisect

def length_of_lis_binary_search(nums: list[int]) -> int:
    """
    LIS using Patience Sorting + Binary Search (lower bound).
    Time Complexity: O(N log N)
    Auxiliary Space: O(N)
    """
    tails = []
    
    for num in nums:
        # Binary search for first element in tails >= num
        idx = bisect.bisect_left(tails, num)
        
        if idx == len(tails):
            tails.append(num) # Extend LIS length
        else:
            tails[idx] = num  # Greedily replace tail with smaller value
            
    return len(tails)
```

---

## 3. Key Algorithmic Variants

### Variant 1: Russian Doll Envelopes (LeetCode 354)

Given 2D envelopes `[w, h]`, find max number of envelopes you can nest inside each other (`w1 < w2` AND `h1 < h2`).

#### Sorting Transformation Trick
1. Sort envelopes by **width ascending**.
2. If widths are equal, sort by **height DESCENDING**!
3. Run standard 1D LIS on heights array!

```text
Why height DESCENDING when widths are equal?
Envelope A: [5, 4], Envelope B: [5, 5]
If sorted ascending: heights = [4, 5] -> LIS would pick BOTH [5,4] and [5,5] (WRONG! Widths are equal!).
If sorted descending: heights = [5, 4] -> LIS picks only ONE! (Correct!).
```

```python
def max_envelopes(envelopes: list[list[int]]) -> int:
    """
    Russian Doll Envelopes using 2D Sorting + 1D LIS Binary Search.
    Time Complexity: O(N log N)
    Auxiliary Space: O(N)
    """
    # Sort width ascending, height descending
    envelopes.sort(key=lambda x: (x[0], -x[1]))
    
    heights = [h for w, h in envelopes]
    
    # 1D LIS on heights
    tails = []
    for h in heights:
        idx = bisect.bisect_left(tails, h)
        if idx == len(tails):
            tails.append(h)
        else:
            tails[idx] = h
            
    return len(tails)
```

---

## 4. Common Pitfalls & Edge Cases

1. **Strictly Increasing vs Non-Decreasing**: For strictly increasing (`<`), use `bisect_left`. For non-decreasing (`<=`), use `bisect_right`!
2. **Confusing `tails` with the actual LIS subsequence**: The `tails` array stores tail candidates for each length, NOT the actual LIS sequence itself! To reconstruct the exact LIS string/array, store predecessor pointers.
3. **Quadratic Timeout TLE**: Running $O(N^2)$ DP when $N = 100,000$ causes Time Limit Exceeded. Always implement $O(N \log N)$ Patience Sorting for large $N$.

---

## 5. Practice Problems

### Foundation
- [LeetCode 300: Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)
- [LeetCode 673: Number of Longest Increasing Subsequence](https://leetcode.com/problems/number-of-longest-increasing-subsequence/)

### Intermediate
- [LeetCode 354: Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/)
- [LeetCode 1671: Minimum Number of Removals to Make Mountain Array](https://leetcode.com/problems/minimum-number-of-removals-to-make-mountain-array/) (Dual LIS)

### Advanced
- [LeetCode 1713: Minimum Operations to Make a Subsequence](https://leetcode.com/problems/minimum-operations-to-make-a-subsequence/) (LCS to LIS Reduction)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Dynamic Programming]]
- [[Binary Search]]
- [[Longest Common Subsequence]]
- [[Sorting Algorithms]]
