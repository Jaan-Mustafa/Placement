# Check for Prime  🟢

**Reference:** [GeeksforGeeks — Primality Test](https://www.geeksforgeeks.org/primality-test-set-1-introduction-and-school-method/)

---

## Problem
Given an integer N, determine whether it is **prime** (exactly two divisors: 1 and itself).

```
Input:  7    Output: true
Input:  1    Output: false   (1 is not prime)
Input:  9    Output: false   (9 = 3 × 3)
```

---

## Intuition
A number N is prime if it has no divisor in the range `[2, √N]`. We only need to check up to √N: if N had a divisor larger than √N, its co-divisor would be smaller than √N and we'd have already found it.

---

## Code (C++)
```cpp
bool isPrime(int n) {
    if (n <= 1) return false;              // 0 and 1 are not prime
    for (int i = 2; i * i <= n; i++) {     // check up to sqrt(n)
        if (n % i == 0) return false;      // found a divisor
    }
    return true;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(√N) |
| Space | O(1) |

> To check primality for **many** numbers up to a limit, precompute with the **Sieve of Eratosthenes** in O(N log log N) — see Step 8 (Advanced Maths).
