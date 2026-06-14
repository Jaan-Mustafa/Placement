# Find the Missing Number  🟢

**Reference:** [LeetCode 268 — Missing Number](https://leetcode.com/problems/missing-number/)

---

## Problem
An array contains N distinct numbers taken from `0..N` (or `1..N`). Exactly one is missing — find it.

```
Input:  [3, 0, 1]       Output: 2
Input:  [0, 1, 2, 4, 5], N=5    Output: 3
```

---

## Intuition
Two classic O(N), O(1) tricks:
1. **Sum formula:** expected sum of `0..N` is `N(N+1)/2`. Subtract the actual array sum → the missing number.
2. **XOR:** XOR all indices `0..N` with all elements. Pairs cancel (`x ^ x = 0`), leaving the missing one. XOR avoids overflow.

---

## Code (C++)
```cpp
// XOR approach — overflow-safe
int missingNumber(vector<int>& nums) {
    int n = nums.size(), x = 0;
    for (int i = 0; i < n; i++) {
        x ^= i;             // xor all indices 0..n-1
        x ^= nums[i];       // xor all elements
    }
    x ^= n;                 // include n itself
    return x;
}

// Sum approach
int missingSum(vector<int>& nums) {
    int n = nums.size();
    long long total = 1LL * n * (n + 1) / 2;
    for (int x : nums) total -= x;
    return (int)total;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Prefer XOR when N is large to avoid integer overflow in the sum.
