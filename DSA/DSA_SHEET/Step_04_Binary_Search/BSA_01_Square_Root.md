# Square Root of a Number  🟡

**Reference:** [LeetCode 69 — Sqrt(x)](https://leetcode.com/problems/sqrtx/)

---

## Problem
Given a non-negative integer N, return the floor of its square root (integer part).

```
Input:  N = 28    Output: 5   (5*5=25 ≤ 28 < 36)
Input:  N = 36    Output: 6
```

---

## Intuition: binary search on the answer
The answer lies in `[1, N]`. We search for the **largest** integer `x` with `x*x <= N`. At each `mid`, if `mid*mid <= N`, `mid` is a candidate — record it and search higher; else search lower. This is "binary search on the answer space" rather than on an array.

---

## Code (C++)
```cpp
int mySqrt(int n) {
    long long low = 1, high = n, ans = 0;
    while (low <= high) {
        long long mid = low + (high - low) / 2;
        if (mid * mid <= n) {
            ans = mid;              // candidate; try a bigger root
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return (int)ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> Use `long long` for `mid * mid` — it overflows `int` for large N. This "search the largest valid value" template recurs across all BS-on-answer problems.
