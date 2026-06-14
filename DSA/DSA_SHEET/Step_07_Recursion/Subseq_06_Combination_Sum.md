# Combination Sum  🟡

**Reference:** [LeetCode 39 — Combination Sum](https://leetcode.com/problems/combination-sum/)

---

## Problem
Given distinct candidates and a target, return all unique combinations summing to target. **Each candidate may be used unlimited times.**

```
Input:  candidates = [2, 3, 6, 7], target = 7
Output: [[2,2,3], [7]]
```

---

## Intuition
Pick/not-pick, but on a "pick" we **stay at the same index** (reuse allowed). When we skip an element, we move to the next index permanently. This avoids duplicate combinations like `[2,3]` and `[3,2]`.

---

## Code (C++)
```cpp
void solve(int i, vector<int>& cand, int target,
           vector<int>& cur, vector<vector<int>>& res) {
    if (i == cand.size()) {
        if (target == 0) res.push_back(cur);
        return;
    }
    if (cand[i] <= target) {                  // PICK (reuse same index)
        cur.push_back(cand[i]);
        solve(i, cand, target - cand[i], cur, res);
        cur.pop_back();
    }
    solve(i + 1, cand, target, cur, res);     // SKIP to next index
}

vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    vector<vector<int>> res;
    vector<int> cur;
    solve(0, candidates, target, cur, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2^target) loose bound |
| Space | O(target / min candidate) recursion depth |

> The key difference from power set: on PICK we recurse with the **same** `i`, enabling unlimited reuse.
