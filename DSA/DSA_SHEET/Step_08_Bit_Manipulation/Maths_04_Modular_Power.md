# Power(a, b) % m — Modular Exponentiation  🟡

**Reference:** [LeetCode 50 — Pow(x, n)](https://leetcode.com/problems/powx-n/) · [GeeksforGeeks — Modular Exponentiation](https://www.geeksforgeeks.org/modular-exponentiation-power-in-modular-arithmetic/)

---

## Problem
Compute `(a^b) mod m` efficiently for large `b` (which is too big to multiply naively `b` times).

```
a = 2, b = 10, m = 1000 → 1024 % 1000 = 24
a = 3, b = 13, m = 7     → 1594323 % 7 = 3
```

---

## Intuition
**Binary (fast) exponentiation.** Write `b` in binary: `a^b` is the product of `a^(2^k)` for every set bit `k` of `b`. Square the base each step (`a, a², a⁴, ...`) and multiply it into the result only when the current lowest bit of `b` is 1. This turns `b` multiplications into `log b`. Take `% m` after every multiply to keep numbers small.

---

## Code (C++)
```cpp
long long modpow(long long a, long long b, long long m) {
    a %= m;                         // normalise base
    long long result = 1 % m;       // handles m == 1 → result 0
    while (b > 0) {
        if (b & 1)                  // current lowest bit of exponent set?
            result = (result * a) % m;
        a = (a * a) % m;            // square the base
        b >>= 1;                    // move to next bit
    }
    return result;
}
```

⚠️ Use `long long` for all of `a`, `result`, and the products: `(result * a)` can reach ~`(m-1)²`, which overflows 32-bit `int` for any `m` above ~46000. For `m` near `10^18` you'd need 128-bit / mulmod tricks.

---

## Complexity
| | |
|---|---|
| Time | O(log b) |
| Space | O(1) |

> This is the workhorse behind modular inverse (Fermat's little theorem) and competitive-programming "answer mod 1e9+7".
