# Merge Overlapping Intervals  🔴

**Reference:** [LeetCode 56 — Merge Intervals](https://leetcode.com/problems/merge-intervals/)

---

## Problem
Given a list of intervals, merge all overlapping ones.

```
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

---

## Intuition
**Sort by start.** Then sweep: keep the last merged interval; if the current interval's start is ≤ the last interval's end, they overlap → extend the end to `max(end, current.end)`. Otherwise it's disjoint → push a new interval.

---

## Code (C++)
```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());     // by start
    vector<vector<int>> res;
    for (auto& iv : intervals) {
        if (res.empty() || res.back()[1] < iv[0]) {
            res.push_back(iv);                     // no overlap -> new interval
        } else {
            res.back()[1] = max(res.back()[1], iv[1]);  // overlap -> extend end
        }
    }
    return res;
}
```

---

## Dry run
Sorted: `[1,3],[2,6],[8,10],[15,18]`. `[1,3]` pushed; `[2,6]` overlaps (2≤3) → `[1,6]`; `[8,10]` disjoint (8>6) → push; `[15,18]` disjoint → push.

## Complexity
| | |
|---|---|
| Time | O(N log N) (sort dominates) |
| Space | O(N) for output (O(1) extra) |

> Sorting by start is essential — it guarantees any overlap is with the immediately preceding merged interval.
