# Search in Rotated Sorted Array II (with Duplicates)  🟡

**Reference:** [LeetCode 81 — Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)

---

## Problem
Same as version I, but the array may contain **duplicates**. Return whether `target` exists (true/false).

```
Input:  [2, 5, 6, 0, 0, 1, 2], target = 0    Output: true
Input:  [2, 5, 6, 0, 0, 1, 2], target = 3    Output: false
```

---

## Intuition
Duplicates break the "one half is sorted" detection when `a[low] == a[mid] == a[high]` — we can't tell which side is sorted. The fix: when this ambiguity occurs, **shrink both ends** (`low++, high--`) and continue. Otherwise, proceed exactly like version I.

---

## Code (C++)
```cpp
bool search(vector<int>& a, int target) {
    int low = 0, high = a.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] == target) return true;

        // ambiguous case: can't determine sorted half
        if (a[low] == a[mid] && a[mid] == a[high]) {
            low++; high--;
            continue;
        }

        if (a[low] <= a[mid]) {                 // left half sorted
            if (a[low] <= target && target < a[mid]) high = mid - 1;
            else low = mid + 1;
        } else {                                // right half sorted
            if (a[mid] < target && target <= a[high]) low = mid + 1;
            else high = mid - 1;
        }
    }
    return false;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) average, **O(N) worst** (e.g. all elements equal) |
| Space | O(1) |

> The `low++, high--` shrink is what degrades the worst case to O(N) — unavoidable with duplicates like `[3,3,3,3,3]`.
