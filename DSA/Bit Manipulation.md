---
title: Bit Manipulation, Bitwise Hacks & Submask Iteration
aliases:
  - Bit Manipulation
  - Bitwise
  - XOR
  - Bitwise Hacks
  - Submask Iteration
tags:
  - dsa
  - bit-manipulation
  - bitwise
status: complete
---

# Bit Manipulation, Bitwise Hacks & Submask Iteration

## TL;DR

**Bit Manipulation** operates directly on the raw binary bit representations of integers using low-level CPU logic gates (`AND`, `OR`, `XOR`, `NOT`, `SHIFTS`). Bitwise operations execute in **1 clock cycle (constant time $O(1)$)** and consume $O(1)$ memory space, offering drastic performance optimizations for sets, state masks, and mathematical problems.

---

## 1. Bitwise Operators Truth Table

| $A$ | $B$ | $A \; \& \; B$ (AND) | $A \mid B$ (OR) | $A \oplus B$ (XOR) | $\sim A$ (NOT 4-bit) |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | **0** | 1111 (15) |
| 0 | 1 | 0 | 1 | **1** | 1110 (14) |
| 1 | 0 | 0 | 1 | **1** | 1101 (13) |
| 1 | 1 | 1 | 1 | **0** | 1100 (12) |

### Key Properties of XOR ($\oplus$)

$$x \oplus x = 0 \quad \text{(Self-cancellation)}$$

$$x \oplus 0 = x \quad \text{(Identity)}$$

$$a \oplus b = b \oplus a \quad \text{(Commutative)}$$

$$(a \oplus b) \oplus c = a \oplus (b \oplus c) \quad \text{(Associative)}$$

---

## 2. The 10 Essential Bitwise Formulas & Hacks

```text
┌────────────────────────────────────────────────────────┐
│ 1. Check if i-th bit is SET (1): (n & (1 << i)) != 0   │
├────────────────────────────────────────────────────────┤
│ 2. SET i-th bit to 1:            n | (1 << i)          │
├────────────────────────────────────────────────────────┤
│ 3. CLEAR i-th bit to 0:          n & ~(1 << i)         │
├────────────────────────────────────────────────────────┤
│ 4. TOGGLE i-th bit:              n ^ (1 << i)          │
├────────────────────────────────────────────────────────┤
│ 5. Clear lowest set bit:         n & (n - 1)           │
├────────────────────────────────────────────────────────┤
│ 6. Extract lowest set bit:       n & (-n)              │
├────────────────────────────────────────────────────────┤
│ 7. Check Power of Two (n > 0):   (n & (n - 1)) == 0    │
├────────────────────────────────────────────────────────┤
│ 8. In-place Swap (a, b):         a^=b; b^=a; a^=b;     │
├────────────────────────────────────────────────────────┤
│ 9. Multiply by 2ᵏ:               n << k                │
├────────────────────────────────────────────────────────┤
│ 10. Floor Divide by 2ᵏ:          n >> k                │
└────────────────────────────────────────────────────────┘
```

---

## 3. Visual Breakdown: Brian Kernighan's Algorithm (`n & (n - 1)`)

Clearing the lowest set bit `n & (n - 1)` drops the rightmost `1` bit in exact $O(\text{Set Bits})$ iterations.

```text
n = 12 (Binary: 1100_2)
n - 1 = 11 (Binary: 1011_2)

  1100  (12)
& 1011  (11)
───────
  1000  (8)  ◄── Lowest set bit at position 2 is CLEARED!
```

---

## 4. Advanced Bitwise Pattern: Submask Iteration ($O(3^N)$ Total)

Given a bitmask `mask` representing a set, iterate over **all non-empty submasks** in descending order.

$$\text{Submask Recurrence}: \text{sub} = (\text{sub} - 1) \; \& \; \text{mask}$$

```text
Submask Iteration for mask = 1011_2 (11):

Step 1: sub = 1011_2 (11)
Step 2: sub = (1011 - 1) & 1011 = 1010 & 1011 = 1010_2 (10)
Step 3: sub = (1010 - 1) & 1011 = 1001 & 1011 = 1001_2 (9)
Step 4: sub = (1001 - 1) & 1011 = 1000 & 1011 = 1000_2 (8)
Step 5: sub = (1000 - 1) & 1011 = 0111 & 1011 = 0011_2 (3)
Step 6: sub = (0011 - 1) & 1011 = 0010 & 1011 = 0010_2 (2)
Step 7: sub = (0010 - 1) & 1011 = 0001 & 1011 = 0001_2 (1)
Step 8: sub = (0001 - 1) & 1011 = 0000 -> Loop Terminates!
```

```python
def iterate_submasks(mask: int) -> list[int]:
    """Iterates all non-empty submasks of a bitmask."""
    submasks = []
    sub = mask
    while sub > 0:
        submasks.append(sub)
        sub = (sub - 1) & mask
    return submasks
```

---

## 5. Complete Python Code Walkthroughs

### Problem 1: Single Number III (Two Unique Elements - LeetCode 260)

Given an integer array `nums` where every element appears twice except for **two numbers** that appear once, find the two unique numbers in $O(N)$ time and $O(1)$ space.

#### Strategy
1. XOR all numbers: `total_xor = a ^ b`.
2. Extract any set bit (e.g. Lowest Set Bit `diff = total_xor & (-total_xor)`).
3. Partition numbers into two groups based on whether `(num & diff) == 0`. `a` and `b` fall into different groups, and all duplicate pairs cancel out!

```python
def single_number_iii(nums: list[int]) -> list[int]:
    """
    Find two unique numbers using LSB bitwise partitioning.
    Time Complexity: O(N)
    Auxiliary Space: O(1)
    """
    total_xor = 0
    for num in nums:
        total_xor ^= num
        
    # Extract lowest set bit (where number 'a' and 'b' differ)
    diff = total_xor & (-total_xor)
    
    a = 0
    b = 0
    for num in nums:
        if num & diff:
            a ^= num
        else:
            b ^= num
            
    return [a, b]
```

---

### Problem 2: Counting Bits ($O(N)$ DP + Bit Trick - LeetCode 338)

Given integer $N$, return an array `ans` of length $N+1$ such that `ans[i]` is the number of 1's in the binary representation of $i$.

```python
def count_bits(n: int) -> list[int]:
    """
    Counting Bits using DP + Right Shift.
    Time Complexity: O(N)
    Auxiliary Space: O(N) output
    """
    ans = [0] * (n + 1)
    for i in range(1, n + 1):
        # bit_count(i) = bit_count(i >> 1) + (i & 1)
        ans[i] = ans[i >> 1] + (i & 1)
    return ans
```

---

## 6. Common Pitfalls & Edge Cases

1. **Operator Precedence Bug**: Bitwise operators (`&`, `|`, `^`) have **lower precedence** than comparison operators (`==`, `!=`, `<`, `>`)!
   - `1 << i - 1` evaluates as `1 << (i - 1)` (WRONG!). Always use explicit parentheses: `(1 << i) - 1`.
   - `n & 1 == 0` evaluates as `n & (1 == 0)` (WRONG!). Always use `(n & 1) == 0`.
2. **32-Bit Signed Integer Overflow in Shift**: Shifting `1 << 31` in C++ with standard signed 32-bit `int` causes undefined behavior overflow. Use `1LL << 31` for 64-bit shifts!
3. **Negative Numbers in Two's Complement**: In Python, integers have infinite precision bits so `~x` returns `-(x + 1)`. To restrict to 32 bits, mask with `0xFFFFFFFF`.

---

## 7. Practice Problems

### Foundation
- [LeetCode 136: Single Number](https://leetcode.com/problems/single-number/)
- [LeetCode 191: Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/)
- [LeetCode 231: Power of Two](https://leetcode.com/problems/power-of-two/)

### Intermediate
- [LeetCode 260: Single Number III](https://leetcode.com/problems/single-number-iii/)
- [LeetCode 338: Counting Bits](https://leetcode.com/problems/counting-bits/)
- [LeetCode 78: Subsets](https://leetcode.com/problems/subsets/) (Bitmask Iteration)
- [LeetCode 201: Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/)

### Advanced
- [LeetCode 847: Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) (BFS + Bitmask State)
- [LeetCode 1494: Parallel Courses II](https://leetcode.com/problems/parallel-courses-ii/) (DP + Submask Iteration)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Complexity Analysis]]
- [[Fenwick Tree]]
- [[Trie]]
- [[Dynamic Programming]]
