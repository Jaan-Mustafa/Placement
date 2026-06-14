# Power Set (All Subsets via Bitmask)  🟡

**Reference:** [LeetCode 78 — Subsets](https://leetcode.com/problems/subsets/) · [GeeksforGeeks — Power Set](https://www.geeksforgeeks.org/power-set/)

---

## Problem
Given a set of `n` distinct elements, generate its **power set** — all `2^n` subsets.

```
Input:  [1, 2, 3]
Output: [], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]
```

---

## Intuition
Each subset corresponds to a binary **mask** of `n` bits: bit `i` set means "include element `i`". There are `2^n` masks (`0 .. 2^n - 1`), so iterating every mask enumerates every subset exactly once. Bit `i` of the mask is checked with `(mask >> i) & 1`.

---

## Code (C++)
```cpp
vector<vector<int>> powerSet(vector<int>& nums) {
    int n = nums.size();
    vector<vector<int>> result;
    int total = 1 << n;                    // 2^n subsets

    for (int mask = 0; mask < total; ++mask) {
        vector<int> subset;
        for (int i = 0; i < n; ++i) {
            if (mask & (1 << i))           // is element i included?
                subset.push_back(nums[i]);
        }
        result.push_back(subset);
    }
    return result;
}
```

⚠️ `1 << n` overflows a 32-bit `int` for `n >= 31`. Power-set problems are only feasible for small `n` (≈ ≤ 20) anyway, since the output itself is `2^n` subsets.

---

## Complexity
| | |
|---|---|
| Time | O(2^n · n) — n bits per mask |
| Space | O(2^n · n) for the output |
