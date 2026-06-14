# Find Peak Element  🟡

**Reference:** [LeetCode 162 — Find Peak Element](https://leetcode.com/problems/find-peak-element/)

---

## Problem
A peak element is strictly greater than its neighbors. Return the index of **any** peak. Assume `a[-1] = a[n] = -∞` (boundaries count as negative infinity). Required: O(log N).

```
Input:  [1, 2, 3, 1]       Output: 2   (3 is a peak)
Input:  [1, 2, 1, 3, 5, 6, 4]    Output: 5 (or 1)
```

---

## Intuition
Binary search on the **slope**. At `mid`:
- If `a[mid] < a[mid+1]`, we're on an ascending slope → a peak lies to the **right** (`low = mid + 1`).
- If `a[mid] > a[mid+1]`, we're descending → a peak is at `mid` or to the **left** (`high = mid`).

Because boundaries are -∞, an ascending slope must eventually turn down, guaranteeing a peak.

---

## Code (C++)
```cpp
int findPeakElement(vector<int>& a) {
    int low = 0, high = a.size() - 1;
    while (low < high) {
        int mid = low + (high - low) / 2;
        if (a[mid] < a[mid + 1])
            low = mid + 1;          // peak to the right
        else
            high = mid;             // peak at mid or to the left
    }
    return low;                     // low == high == a peak index
}
```

---

## Why does halving work without full sorting?
We don't need the global max — any local peak works. Following the upward slope always leads to a peak, so discarding the "downhill" half is safe.

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |
