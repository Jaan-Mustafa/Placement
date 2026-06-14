# Count Good Numbers  🟡

**Reference:** [LeetCode 1922 — Count Good Numbers](https://leetcode.com/problems/count-good-numbers/)

---

## Problem
A digit string is **good** if digits at **even** indices (0-indexed) are even (0,2,4,6,8 → 5 choices) and digits at **odd** indices are prime (2,3,5,7 → 4 choices). Count good strings of length `n`, modulo 1e9+7.

```
Input:  n = 1    Output: 5
Input:  n = 4    Output: 400
```

---

## Intuition
Of the `n` positions, `ceil(n/2)` are even-indexed (5 choices each) and `floor(n/2)` are odd-indexed (4 choices each). The answer is `5^ceil(n/2) * 4^floor(n/2)`. Use **fast modular exponentiation** (recursive binary power with mod) since `n` is huge.

---

## Code (C++)
```cpp
const int MOD = 1e9 + 7;

long long power(long long x, long long n) {
    if (n == 0) return 1;
    long long half = power(x, n / 2) % MOD;
    long long res = half * half % MOD;
    if (n % 2) res = res * x % MOD;
    return res;
}

int countGoodNumbers(long long n) {
    long long even = (n + 1) / 2;       // even indices: positions 0,2,4...
    long long odd  = n / 2;             // odd indices
    return (int)(power(5, even) * power(4, odd) % MOD);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log n) |
| Space | O(log n) recursion |

> Take `% MOD` after **every** multiply to prevent overflow. This is fast power applied to a combinatorics count.
