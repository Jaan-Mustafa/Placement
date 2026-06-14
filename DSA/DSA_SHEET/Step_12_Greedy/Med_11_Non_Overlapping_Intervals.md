# Non-overlapping Intervals  🟡

**Reference:** [LeetCode 435 — Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)

---

## Problem
Given an array of intervals, return the **minimum number of intervals to remove** so the rest are non-overlapping.

```
Input:  [[1,2],[2,3],[3,4],[1,3]]
Output: 1   (remove [1,3])
```

---

## Intuition
This is the dual of **activity selection**: maximize the number of intervals you keep, then removals = total − kept. Sort by **end time**. Greedily keep an interval if its start is `≥` the end of the last kept interval (no overlap), updating the boundary. Any interval that overlaps the last kept one must be removed — choosing the earliest-ending interval to keep leaves maximum room for the rest.

---

## Code (C++)
```cpp
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;
    sort(intervals.begin(), intervals.end(),
         [](const vector<int>& a, const vector<int>& b){ return a[1] < b[1]; }); // by end

    int removals = 0;
    int lastEnd = intervals[0][1];     // first interval is always kept
    for (int i = 1; i < (int)intervals.size(); i++) {
        if (intervals[i][0] >= lastEnd) {
            lastEnd = intervals[i][1]; // keep it, advance boundary
        } else {
            removals++;                // overlaps -> remove this one
        }
    }
    return removals;
}
```

---

## Dry run
Sorted by end: [1,2],[2,3],[1,3],[3,4]. Keep [1,2] (end2); [2,3] start2≥2 keep (end3); [1,3] start1<3 remove (removals=1); [3,4] start3≥3 keep. Answer **1**.

## Complexity
| | |
|---|---|
| Time | O(N log N) (sorting) |
| Space | O(1) extra |

> ⚠️ Sort by **end** time, not start — that's what makes the greedy optimal here. Use `>=` so intervals that merely touch endpoints (e.g. `[1,2],[2,3]`) are considered non-overlapping.
