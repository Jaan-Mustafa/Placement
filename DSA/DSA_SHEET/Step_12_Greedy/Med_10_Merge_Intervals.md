# Merge Intervals  🟡

**Reference:** [LeetCode 56 — Merge Intervals](https://leetcode.com/problems/merge-intervals/)

---

## Problem
Given an array of intervals, merge all overlapping intervals and return the non-overlapping result.

```
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

---

## Intuition
**Sort by start time.** Then sweep once: keep the last interval in the result; if the current interval's start is `≤` the last result's end, they overlap → extend the end to `max(end, current.end)`. Otherwise it's disjoint → push it as a new interval. Sorting by start guarantees any overlap is with the immediately preceding merged interval.

---

## Code (C++)
```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());   // by start time
    vector<vector<int>> res;
    for (auto& iv : intervals) {
        if (res.empty() || res.back()[1] < iv[0]) {
            res.push_back(iv);                   // disjoint -> new interval
        } else {
            res.back()[1] = max(res.back()[1], iv[1]);  // overlap -> extend end
        }
    }
    return res;
}
```

---

## Dry run
Sorted: `[1,3],[2,6],[8,10],[15,18]`. Push [1,3]; [2,6] overlaps (2≤3) → [1,6]; [8,10] disjoint (8>6) → push; [15,18] disjoint → push. Result **[[1,6],[8,10],[15,18]]**.

## Complexity
| | |
|---|---|
| Time | O(N log N) (sort dominates) |
| Space | O(N) for output (O(1) extra) |

> ⚠️ Sorting by start is essential. Use `res.back()[1] < iv[0]` for the disjoint test; switch to `<=` only if touching intervals like `[1,2],[2,3]` should NOT merge.
