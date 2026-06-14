# Factorial of N (Recursion)  🟢

**Reference:** [GeeksforGeeks — Factorial using recursion](https://www.geeksforgeeks.org/program-for-factorial-of-a-number/)

---

## Problem
Compute N! = 1 × 2 × ... × N using recursion.

```
Input:  N = 5    Output: 120   (5×4×3×2×1)
Input:  N = 0    Output: 1     (0! = 1 by definition)
```

---

## Intuition
`fact(N) = N × fact(N-1)`, with base case `fact(0) = 1`.

---

## Code (C++)
```cpp
long long factorial(int n) {
    if (n <= 1) return 1;              // 0! = 1! = 1
    return (long long)n * factorial(n - 1);
}
```

---

## Recursion unwinding (N=4)
```
fact(4) = 4 * fact(3)
        = 4 * (3 * fact(2))
        = 4 * (3 * (2 * fact(1)))
        = 4 * 3 * 2 * 1 = 24
```

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |

> Use `long long` — factorials overflow `int` past 12! and `long long` past 20!.
