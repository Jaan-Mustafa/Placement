# Nth Root of a Number  🟡

**Reference:** [GeeksforGeeks — Nth root](https://www.geeksforgeeks.org/n-th-root-number/)

---

## Problem
Find the integer `x` such that `xⁿ = m`. If no integer root exists, return -1.

```
Input:  n = 3, m = 27    Output: 3   (3³ = 27)
Input:  n = 4, m = 69    Output: -1  (no integer 4th root)
```

---

## Intuition
Binary search on the answer in `[1, m]`. For a candidate `mid`, compute `midⁿ` (carefully, stopping early if it exceeds `m` to avoid overflow). Compare to `m`: equal → answer; less → go right; greater → go left.

---

## Code (C++)
```cpp
// returns 1 if x^n < m, 0 if ==, 2 if > m  (early-exit to avoid overflow)
int power(long long x, int n, long long m) {
    long long res = 1;
    for (int i = 0; i < n; i++) {
        res *= x;
        if (res > m) return 2;          // too big, stop early
    }
    return res == m ? 0 : 1;
}

int nthRoot(int n, long long m) {
    long long low = 1, high = m;
    while (low <= high) {
        long long mid = low + (high - low) / 2;
        int cmp = power(mid, n, m);
        if (cmp == 0) return (int)mid;  // exact root
        else if (cmp == 1) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n · log m) — log m BS steps, each computing a power in O(n) |
| Space | O(1) |

> The early-exit in `power` is essential: `midⁿ` overflows fast, so bail out the moment it passes `m`.
