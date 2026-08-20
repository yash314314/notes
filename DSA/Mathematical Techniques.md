---
title: Mathematical Techniques, Number Theory & Modular Arithmetic
aliases:
  - Mathematical Techniques
  - Number Theory
  - Modular Arithmetic
  - Sieve of Eratosthenes
  - Euclidean Algorithm
  - Fermat's Little Theorem
tags:
  - dsa
  - math
  - number-theory
status: complete
---

# Mathematical Techniques, Number Theory & Modular Arithmetic

## TL;DR

Numerical algorithms rely heavily on foundational **Number Theory** and **Modular Arithmetic**. Key routines include finding the **Greatest Common Divisor ($\gcd$)** in $O(\log(\min(A, B)))$ time using the Euclidean Algorithm, precomputing primes up to $N$ in **$O(N \log \log N)$** time using the Sieve of Eratosthenes, and computing $a^b \pmod M$ in **$O(\log B)$** via Binary Modular Exponentiation.

---

## 1. Euclidean Algorithm for GCD & LCM

The **Greatest Common Divisor ($\gcd$)** of two non-zero integers $a$ and $b$ is the largest positive integer that divides both $a$ and $b$ without remainder.

### Euclidean Recurrence Relation

$$\gcd(a, b) = \begin{cases} a & \text{if } b = 0 \\ \gcd(b, \; a \pmod b) & \text{if } b > 0 \end{cases}$$

### Least Common Multiple (LCM) Formula

$$\text{lcm}(a, b) = \frac{|a \cdot b|}{\gcd(a, b)}$$

```text
Euclidean Tracing for gcd(48, 18):
Step 1: 48 mod 18 = 12  ──► gcd(18, 12)
Step 2: 18 mod 12 = 6   ──► gcd(12, 6)
Step 3: 12 mod 6  = 0   ──► gcd(6, 0)
Step 4: b == 0          ──► Returns 6!
```

#### Python Implementation

```python
def gcd(a: int, b: int) -> int:
    """Euclidean Algorithm for GCD. Time: O(log(min(A, B)))."""
    while b:
        a, b = b, a % b
    return a

def lcm(a: int, b: int) -> int:
    """LCM calculation avoiding overflow by dividing before multiplying."""
    if a == 0 or b == 0:
        return 0
    return (a // gcd(a, b)) * b
```

---

## 2. Primality Testing & Sieve of Eratosthenes

### 1. Single Number Trial Division ($O(\sqrt{N})$)
If a number $N$ is composite, it must have a prime factor $\le \sqrt{N}$.

```python
def is_prime(n: int) -> bool:
    """Trial Division Primality Test. Time: O(√N)."""
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0 or n % 3 == 0:
        return False
        
    i = 5
    while i * i <= n:
        if n % i == 0 or n % (i + 2) == 0:
            return False
        i += 6
        
    return True
```

---

### 2. Sieve of Eratosthenes ($O(N \log \log N)$ Batch Generation)

Generates all prime numbers up to $N$ by iteratively marking multiples of each prime starting from $p^2$.

```text
Sieve Visual Sweeping up to N = 10:
Initial: [2, 3, 4, 5, 6, 7, 8, 9, 10]
p = 2: Mark multiples 4, 6, 8, 10 as False
p = 3: Mark multiples 9 as False
Primes Remaining: [2, 3, 5, 7]
```

#### Python Implementation

```python
def sieve_of_eratosthenes(n: int) -> list[int]:
    """
    Sieve of Eratosthenes.
    Time Complexity: O(N log log N)
    Auxiliary Space: O(N)
    """
    if n < 2:
        return []
        
    is_prime_flag = [True] * (n + 1)
    is_prime_flag[0] = is_prime_flag[1] = False
    
    p = 2
    while p * p <= n:
        if is_prime_flag[p]:
            # Mark multiples starting from p²
            for i in range(p * p, n + 1, p):
                is_prime_flag[i] = False
        p += 1
        
    return [i for i in range(2, n + 1) if is_prime_flag[i]]
```

---

## 3. Modular Arithmetic & Fast Binary Exponentiation

In competitive programming, results are often requested modulo $M = 10^9 + 7$.

### Fundamental Modular Identity Laws

$$(A + B) \pmod M = ((A \pmod M) + (B \pmod M)) \pmod M$$

$$(A \cdot B) \pmod M = ((A \pmod M) \cdot (B \pmod M)) \pmod M$$

$$(A - B) \pmod M = ((A \pmod M) - (B \pmod M) + M) \pmod M$$

---

### Fast Binary Exponentiation ($a^b \pmod M$ in $O(\log B)$)

$$\text{pow}(a, b, M) = \begin{cases} 1 & \text{if } b = 0 \\ \left( \text{pow}(a, b/2, M) \right)^2 \pmod M & \text{if } b \text{ is even} \\ a \cdot \left( \text{pow}(a, (b-1)/2, M) \right)^2 \pmod M & \text{if } b \text{ is odd} \end{cases}$$

#### Python Implementation

```python
def mod_pow(base: int, exp: int, mod: int) -> int:
    """
    Binary Exponentiation modulo M.
    Time Complexity: O(log EXP)
    Auxiliary Space: O(1)
    """
    result = 1
    base %= mod
    
    while exp > 0:
        if exp & 1:
            result = (result * base) % mod
        base = (base * base) % mod
        exp >>= 1
        
    return result
```

---

### Fermat's Little Theorem for Modular Inverse

Dividing modulo $M$ ($(A / B) \pmod M$) is NOT $(A \pmod M) / (B \pmod M)$. Instead, we multiply by the **Modular Multiplicative Inverse** $B^{-1}$:

$$\left( \frac{A}{B} \right) \pmod M = (A \cdot B^{-1}) \pmod M$$

#### Fermat's Little Theorem
If $M$ is a prime number, then for any integer $B$ not divisible by $M$:

$$B^{M-1} \equiv 1 \pmod M \implies B^{-1} \equiv B^{M-2} \pmod M$$

```python
def mod_inverse(b: int, m: int = 10**9 + 7) -> int:
    """Computes Modular Inverse B⁻¹ mod M using Fermat's Little Theorem."""
    return mod_pow(b, m - 2, m)
```

---

## 4. Combinatorics & Pascal's Triangle ($nCr$)

To calculate Combinations $\binom{n}{r} = \frac{n!}{r!(n-r)!} \pmod M$:
1. Precompute factorials `fact[i]` and inverse factorials `inv_fact[i]` in $O(N)$ time.
2. Answer any $nCr$ query in **$O(1)$ constant time**:

$$\binom{n}{r} \pmod M = \text{fact}[n] \cdot \text{inv\_fact}[r] \cdot \text{inv\_fact}[n-r] \pmod M$$

```python
class Combinatorics:
    def __init__(self, max_n: int, mod: int = 10**9 + 7):
        self.mod = mod
        self.fact = [1] * (max_n + 1)
        self.inv_fact = [1] * (max_n + 1)
        
        for i in range(1, max_n + 1):
            self.fact[i] = (self.fact[i-1] * i) % self.mod
            
        self.inv_fact[max_n] = mod_inverse(self.fact[max_n], self.mod)
        for i in range(max_n - 1, -1, -1):
            self.inv_fact[i] = (self.inv_fact[i+1] * (i + 1)) % self.mod

    def nCr(self, n: int, r: int) -> int:
        if r < 0 or r > n:
            return 0
        return (self.fact[n] * self.inv_fact[r] % self.mod) * self.inv_fact[n-r] % self.mod
```

---

## 5. Common Pitfalls & Edge Cases

1. **Integer Overflow in LCM**: In C++, calculating `(a * b) / gcd(a, b)` can overflow 64-bit integers before division occurs! **Always divide first**: `(a / gcd(a, b)) * b`.
2. **Negative Modulo Result**: In C++, `(-5) % 3` returns `-2`. Always enforce non-negative mod: `(val % M + M) % M`.
3. **Fermat's Theorem Modulus Restriction**: Fermat's Little Theorem ($B^{M-2}$) REQUIRES $M$ to be a **PRIME**. If $M$ is composite, use the **Extended Euclidean Algorithm** to find $B^{-1}$.

---

## 6. Practice Problems

### Foundation
- [LeetCode 1979: Find Greatest Common Divisor of Array](https://leetcode.com/problems/find-greatest-common-divisor-of-array/)
- [LeetCode 204: Count Primes](https://leetcode.com/problems/count-primes/) (Sieve of Eratosthenes)

### Intermediate
- [LeetCode 50: Pow(x, n)](https://leetcode.com/problems/powx-n/)
- [LeetCode 62: Unique Paths](https://leetcode.com/problems/unique-paths/) (Combinatorics $nCr$)
- [LeetCode 858: Mirror Reflection](https://leetcode.com/problems/mirror-reflection/) (LCM Geometry)

### Advanced
- [LeetCode 920: Number of Music Playlists](https://leetcode.com/problems/number-of-music-playlists/) (DP + Combinatorics)
- [LeetCode 2514: Count Anagrams](https://leetcode.com/problems/count-anagrams/) (Modular Inverse $nCr$)

---

## Related Concepts
- [[00 - DSA MOC|Master DSA MOC]]
- [[Complexity Analysis]]
- [[Bit Manipulation]]
- [[Recursion]]
- [[Dynamic Programming]]
