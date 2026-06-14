# Print All Subsequences / Power Set  🟡

**Reference:** [LeetCode 78 — Subsets](https://leetcode.com/problems/subsets/)

---

## Problem
Generate all subsequences (the power set) of an array of distinct integers.

```
Input:  [1, 2, 3]
Output: [], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]   (2³ = 8 subsets)
```

---

## Intuition: pick / don't-pick
For each index, branch two ways: **include** the element or **exclude** it. At the end of the array, record the current subset. This pick/not-pick tree is THE fundamental recursion pattern.

---

## Code (C++)
```cpp
void solve(int i, vector<int>& nums, vector<int>& cur, vector<vector<int>>& res) {
    if (i == nums.size()) { res.push_back(cur); return; }

    cur.push_back(nums[i]);           // PICK nums[i]
    solve(i + 1, nums, cur, res);
    cur.pop_back();                   // backtrack

    solve(i + 1, nums, cur, res);     // DON'T pick nums[i]
}

vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> res;
    vector<int> cur;
    solve(0, nums, cur, res);
    return res;
}
```

### Bitmask alternative
```cpp
for (int mask = 0; mask < (1 << n); mask++) {
    vector<int> sub;
    for (int i = 0; i < n; i++) if (mask & (1 << i)) sub.push_back(nums[i]);
    res.push_back(sub);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2ⁿ · n) |
| Space | O(n) recursion (excluding output) |

> The `pop_back()` after the PICK branch is **backtracking** — it restores state before exploring the next choice.
