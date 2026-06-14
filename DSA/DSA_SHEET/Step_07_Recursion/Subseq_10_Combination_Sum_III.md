# Combination Sum III  🟡

**Reference:** [LeetCode 216 — Combination Sum III](https://leetcode.com/problems/combination-sum-iii/)

---

## Problem
Find all combinations of **k** numbers from **1–9** (each used at most once) that sum to **n**.

```
Input:  k = 3, n = 7    Output: [[1,2,4]]
Input:  k = 3, n = 9    Output: [[1,2,6], [1,3,5], [2,3,4]]
```

---

## Intuition
Loop-based recursion over digits 1–9, starting from `start` to avoid reusing/reordering. Two stopping conditions: the combination has exactly `k` numbers **and** the remaining target is 0 → record it. Prune when the running size exceeds `k` or the digit exceeds the remaining target.

---

## Code (C++)
```cpp
void solve(int start, int k, int target,
           vector<int>& cur, vector<vector<int>>& res) {
    if (cur.size() == k) {
        if (target == 0) res.push_back(cur);
        return;
    }
    for (int d = start; d <= 9; d++) {
        if (d > target) break;                 // sorted digits → prune
        cur.push_back(d);
        solve(d + 1, k, target - d, cur, res); // next digit, used once
        cur.pop_back();
    }
}

vector<vector<int>> combinationSum3(int k, int n) {
    vector<vector<int>> res;
    vector<int> cur;
    solve(1, k, n, cur, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(C(9, k) · k) — limited to 9 digits |
| Space | O(k) recursion |

> Two constraints at once (count = k AND sum = n) — both must hold at the base case.
