# Minimum in Rotated Sorted Array  🟡

**Reference:** [LeetCode 153 — Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

---

## Problem
Find the minimum element of a rotated sorted array of distinct values in O(log N).

```
Input:  [4, 5, 6, 7, 0, 1, 2]    Output: 0
Input:  [3, 4, 5, 1, 2]          Output: 1
```

---

## Intuition
At each step, one half is sorted. The minimum lies in the **unsorted** half (the one containing the rotation pivot). When the left half `[low..mid]` is sorted, `a[low]` is the smallest there — record it, then move to the right half. Otherwise the minimum is in the left half.

---

## Code (C++)
```cpp
int findMin(vector<int>& a) {
    int low = 0, high = a.size() - 1;
    int ans = INT_MAX;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[low] <= a[mid]) {             // left half sorted
            ans = min(ans, a[low]);         // its smallest is a[low]
            low = mid + 1;                  // min must be on the right
        } else {                            // right half sorted
            ans = min(ans, a[mid]);
            high = mid - 1;                 // min is in the left part
        }
    }
    return ans;
}
```

---

## Optimization
If `a[low] <= a[high]`, the current segment is already fully sorted → `a[low]` is its min; you can break early.

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |
