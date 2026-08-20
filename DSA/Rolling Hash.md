---
title: Rolling Hash, Rabin-Karp & Double Hashing Mitigations
aliases:
  - Rolling Hash
  - Rabin-Karp
  - Polynomial Rolling Hash
  - Double Hashing
tags:
  - dsa
  - string-algorithms
  - rolling-hash
status: complete
---

# Rolling Hash, Rabin-Karp & Double Hashing Mitigations

## TL;DR

A **Rolling Hash** (commonly known as the **Rabin-Karp Algorithm**) is a hash function designed to allow fast $O(1)$ hash re-computation when sliding a fixed-size window over a text string.

By converting a string into a **Polynomial Hash Integer**, updating the window hash as it shifts right takes **$O(1)$ time**, enabling string pattern matching in **$O(N + M)$ average time**.

---

## 1. Polynomial Rolling Hash Formula

For a string $S$ of length $L$, base $B$ (typically $B \ge \text{alphabet size}$, e.g., $B = 31$ or $257$), and prime modulus $M$ (e.g., $M = 10^9 + 7$):

$$H(S) = \left( \sum_{i=0}^{L-1} S[i] \cdot B^{L - 1 - i} \right) \pmod M$$

```text
Example Polynomial Hash for S = "cat" (ASCII: c=99, a=97, t=116, Base B=31):

H("cat") = (99 · 31² + 97 · 31¹ + 116 · 31⁰) mod M
         = (99 · 961 + 97 · 31 + 116) mod M
```

---

## 2. $O(1)$ Window Update Transformation Equation

When sliding a window of size $L$ from left to right:
- **Evict** the outgoing character $S[\text{left}]$ at the high-order power position.
- **Add** the incoming character $S[\text{right}]$ at the low-order power position.

$$H_{\text{new}} = \left( \left( H_{\text{old}} - S[\text{left}] \cdot B^{L-1} \right) \cdot B + S[\text{right}] \right) \pmod M$$

```text
Sliding Window Hash Transition:

Window 1: [ c  a  t ] e
           ▲        
           Evict c (subtract 99 · 31²)

Multiply remaining [a t] by B (31): (a · 31¹ + t · 31⁰) · 31 = a · 31² + t · 31¹

Window 2: c [ a  t  e ]
                    ▲
                    Add incoming e (101)

Final H("ate") computed in O(1) time!
```

---

## 3. Mitigating Collisions: Double Hashing

A single hash function with modulus $M \approx 10^9$ has a collision probability of $\approx 1/M$. By the Birthday Paradox, comparing $10^5$ substrings introduces a **high collision risk**!

### Double Hashing Solution
Calculate two independent polynomial hashes using different prime bases $(B_1, B_2)$ and moduli $(M_1, M_2)$:

$$\text{DoubleHash}(S) = \left( H_{B_1, M_1}(S), \;\; H_{B_2, M_2}(S) \right)$$

- Base 1: $B_1 = 31, \; M_1 = 10^9 + 7$
- Base 2: $B_2 = 37, \; M_2 = 10^9 + 9$

$$\text{Collision Probability} = \frac{1}{M_1 \cdot M_2} \approx 10^{-18} \quad \text{(Negligible! Zero False Positives)}$$

---

## 4. $O(1)$ Substring Range Hash Queries via Prefix Hashes

Precompute prefix hashes $P[i]$ and powers of base $B$:

$$P[i] = \left( \sum_{k=0}^{i-1} S[k] \cdot B^{i - 1 - k} \right) \pmod M$$

To query the hash of any substring $S[L \dots R]$ in **$O(1)$ time**:

$$\text{Hash}(S[L \dots R]) = \left( P[R+1] - P[L] \cdot B^{R - L + 1} \right) \pmod M$$

---

## 5. Complete Python Implementation (Rabin-Karp & String Hash)

```python
class StringHash:
    """
    Double Polynomial Rolling Hash for O(1) Substring Queries.
    Precomputation: O(N)
    Query: O(1)
    Space: O(N)
    """
    def __init__(self, s: str):
        self.s = s
        self.n = len(s)
        
        self.B1, self.M1 = 31, 10**9 + 7
        self.B2, self.M2 = 37, 10**9 + 9
        
        self.P1 = [0] * (self.n + 1)
        self.P2 = [0] * (self.n + 1)
        self.pow1 = [1] * (self.n + 1)
        self.pow2 = [1] * (self.n + 1)
        
        for i in range(self.n):
            val = ord(s[i]) - ord('a') + 1
            self.P1[i+1] = (self.P1[i] * self.B1 + val) % self.M1
            self.P2[i+1] = (self.P2[i] * self.B2 + val) % self.M2
            self.pow1[i+1] = (self.pow1[i] * self.B1) % self.M1
            self.pow2[i+1] = (self.pow2[i] * self.B2) % self.M2

    def query(self, l: int, r: int) -> tuple[int, int]:
        """Returns double hash tuple for substring S[l..r] (0-based)."""
        h1 = (self.P1[r+1] - self.P1[l] * self.pow1[r - l + 1]) % self.M1
        h2 = (self.P2[r+1] - self.P2[l] * self.pow2[r - l + 1]) % self.M2
        return (h1, h2)

def rabin_karp(text: str, pattern: str) -> list[int]:
    """
    Rabin-Karp String Matching Algorithm.
    Time Complexity: O(N + M) average
    Auxiliary Space: O(1)
    """
    n, m = len(text), len(pattern)
    if m > n or m == 0:
        return []
        
    B, M = 257, 10**9 + 7
    pattern_hash = 0
    window_hash = 0
    highest_power = pow(B, m - 1, M)
    
    # 1. Compute initial hash for pattern and first window
    for i in range(m):
        pattern_hash = (pattern_hash * B + ord(pattern[i])) % M
        window_hash = (window_hash * B + ord(text[i])) % M
        
    matches = []
    
    # 2. Slide window across text
    for i in range(n - m + 1):
        if window_hash == pattern_hash:
            # Verification step to eliminate false positives
            if text[i : i + m] == pattern:
                matches.append(i)
                
        if i < n - m:
            # Evict text[i], add text[i + m]
            window_hash = (window_hash - ord(text[i]) * highest_power) % M
            window_hash = (window_hash * B + ord(text[i + m])) % M
            
    return matches
```

---

## 6. Common Pitfalls & Edge Cases

1. **Negative Modulo in C++/Python**: In C++, `(A - B) % M` can yield negative results. Always add $M$: `((A - B) % M + M) % M`.
2. **Single Hash Anti-Hash Tests**: In competitive programming, single modulo $10^9 + 7$ with base 31 is vulnerable to adversarial test cases designed to force collisions ($O(N^2)$ worst case). **Always use Double Hashing** or a random base!
3. **Integer Overflow during Multiplication**: In C++, `window_hash * B` overflows standard 32-bit signed integers. Use 64-bit `long long` integers for intermediate calculations.

---

## 7. Practice Problems

### Foundation
- [LeetCode 28: Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/)
- [LeetCode 187: Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/)

### Intermediate
- [LeetCode 1044: Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) (Binary Search + Rolling Hash)
- [LeetCode 1316: Distinct Echo Substrings](https://leetcode.com/problems/distinct-echo-substrings/)

### Advanced
- [LeetCode 214: Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/)
- [LeetCode 1923: Longest Common Subpath](https://leetcode.com/problems/longest-common-subpath/)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Strings]]
- [[Hashing]]
- [[KMP Algorithm]]
- [[Z Algorithm]]
