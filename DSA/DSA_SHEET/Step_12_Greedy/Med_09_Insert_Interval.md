# Insert Interval  🟡

**Reference:** [LeetCode 57 — Insert Interval](https://leetcode.com/problems/insert-interval/)

---

## Problem
Given a list of **non-overlapping**, sorted `intervals` and a `newInterval`, insert it and merge any overlaps so the result stays sorted and non-overlapping.

```
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
```

---

## Intuition
Since the input is already sorted, do a single pass in three phases:
1. **Before:** push all intervals that end before `newInterval` starts (no overlap).
2. **Overlap:** while an interval starts at or before `newInterval`'s end, absorb it into `newInterval` by taking min start / max end. Push the merged interval once.
3. **After:** push all remaining intervals unchanged.

No sorting is needed — the greedy left-to-right sweep exploits the pre-sorted order.

---

## Code (C++)
```cpp
vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
    vector<vector<int>> res;
    int i = 0, n = intervals.size();

    // 1. intervals strictly before newInterval
    while (i < n && intervals[i][1] < newInterval[0])
        res.push_back(intervals[i++]);

    // 2. merge all overlapping intervals into newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        i++;
    }
    res.push_back(newInterval);        // push the merged interval once

    // 3. intervals strictly after
    while (i < n) res.push_back(intervals[i++]);

    return res;
}
```

---

## Dry run
`[[1,3],[6,9]]`, new `[2,5]`: phase 1 pushes nothing (3 ≥ 2). Phase 2: `[1,3]` overlaps (1≤5) → new=[1,5]; `[6,9]` (6>5) stop → push [1,5]. Phase 3: push [6,9]. Result **[[1,5],[6,9]]**.

## Complexity
| | |
|---|---|
| Time | O(N) (single pass) |
| Space | O(N) for output |

> ⚠️ Use `<` in phase 1 and `<=` in phase 2 to treat touching intervals (e.g. `[1,2]` and `[2,3]`) as overlapping. Adjust if the problem treats touching endpoints as disjoint.
