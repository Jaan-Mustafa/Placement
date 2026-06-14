# Combination Sum II  🟡

**Reference:** [LeetCode 40 — Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

---

## Problem
Given candidates (may contain **duplicates**) and a target, return all unique combinations summing to target. **Each number may be used at most once**, and the result must not contain duplicate combinations.

```
Input:  candidates = [10,1,2,7,6,1,5], target = 8
Output: [[1,1,6], [1,2,5], [1,7], [2,6]]
```

---

## Intuition
**Sort** so duplicates are adjacent. Use a loop-based recursion starting at index `idx`. Within the loop, skip a candidate if it equals the previous one at the same recursion level (`j > idx && cand[j] == cand[j-1]`) — this prevents duplicate combinations. Each chosen element recurses with `j + 1` (used once).

---

## Code (C++)
```cpp
void solve(int idx, vector<int>& cand, int target,
           vector<int>& cur, vector<vector<int>>& res) {
    if (target == 0) { res.push_back(cur); return; }
    for (int j = idx; j < cand.size(); j++) {
        if (j > idx && cand[j] == cand[j - 1]) continue;  // skip duplicates
        if (cand[j] > target) break;                      // sorted → no point continuing
        cur.push_back(cand[j]);
        solve(j + 1, cand, target - cand[j], cur, res);   // each used once
        cur.pop_back();
    }
}

vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end());
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
| Time | O(2ⁿ · k) |
| Space | O(n) recursion |

> The `j > idx && cand[j]==cand[j-1]` check is the standard "skip duplicates at the same level" trick — used in Subsets II and Permutations II as well.
