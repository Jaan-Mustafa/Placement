# Pow(x, n)  🟡

**Reference:** [LeetCode 50 — Pow(x, n)](https://leetcode.com/problems/powx-n/)

---

## Problem
Implement `pow(x, n)` (x raised to the power n), where n can be negative.

```
Input:  x = 2.0, n = 10      Output: 1024.0
Input:  x = 2.0, n = -2      Output: 0.25
```

---

## Intuition: binary exponentiation
Naive multiplication is O(n). Instead use **fast power**: `x^n = (x^(n/2))²` for even n, and `x · x^(n-1)` for odd n. This halves the exponent each step → O(log n). For negative n, compute `1 / x^(-n)`.

---

## Code (C++)
```cpp
double power(double x, long long n) {
    if (n == 0) return 1.0;                  // base case
    double half = power(x, n / 2);
    if (n % 2 == 0) return half * half;      // even: (x^(n/2))^2
    else return half * half * x;             // odd: extra x
}

double myPow(double x, int n) {
    long long N = n;                         // promote to avoid INT_MIN overflow
    if (N < 0) return 1.0 / power(x, -N);
    return power(x, N);
}
```

---

## Why `long long N`?
`n` can be `INT_MIN` (-2³¹); negating it as `int` overflows. Promoting to `long long` makes `-N` safe.

## Complexity
| | |
|---|---|
| Time | O(log n) |
| Space | O(log n) recursion |
