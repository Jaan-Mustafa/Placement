# Subset Sum II (Subsets II — Unique Subsets)  🟡

**Reference:** [LeetCode 90 — Subsets II](https://leetcode.com/problems/subsets-ii/)

---

## Problem
Given an array that may contain **duplicates**, return all **unique** subsets.

```
Input:  [1, 2, 2]
Output: [[], [1], [1,2], [1,2,2], [2], [2,2]]
```

---

## Intuition
**Sort** the array, then use loop-based recursion. At each level, the first occurrence of a value is taken; subsequent duplicates at the **same level** are skipped (`j > idx && a[j] == a[j-1]`). Every node of the recursion tree is itself a valid subset, so push `cur` at the start of each call.

---

## Code (C++)
```cpp
void solve(int idx, vector<int>& a, vector<int>& cur, vector<vector<int>>& res) {
    res.push_back(cur);                        // every node is a subset
    for (int j = idx; j < a.size(); j++) {
        if (j > idx && a[j] == a[j - 1]) continue;  // skip duplicate at this level
        cur.push_back(a[j]);
        solve(j + 1, a, cur, res);
        cur.pop_back();
    }
}

vector<vector<int>> subsetsWithDup(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> res;
    vector<int> cur;
    solve(0, nums, cur, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2ⁿ · n) |
| Space | O(n) recursion |

> Sorting + the same-level skip is the universal recipe for "unique subsets/combinations/permutations with duplicates."
