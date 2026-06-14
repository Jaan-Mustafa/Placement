# Search in Rotated Sorted Array I  🟡

**Reference:** [LeetCode 33 — Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)

---

## Problem
A sorted array of **distinct** values is rotated at an unknown pivot. Find the index of `target` in O(log N). Return -1 if absent.

```
Input:  [4, 5, 6, 7, 0, 1, 2], target = 0    Output: 4
Input:  [4, 5, 6, 7, 0, 1, 2], target = 3    Output: -1
```

---

## Intuition
At any `mid`, **one half is always sorted**. Identify which:
- If `a[low] <= a[mid]` → left half is sorted. Check if target lies in `[a[low], a[mid])`; if yes go left, else go right.
- Otherwise the right half is sorted. Check if target lies in `(a[mid], a[high]]`; if yes go right, else go left.

This lets us discard half each step despite the rotation.

---

## Code (C++)
```cpp
int search(vector<int>& a, int target) {
    int low = 0, high = a.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] == target) return mid;

        if (a[low] <= a[mid]) {                 // left half sorted
            if (a[low] <= target && target < a[mid]) high = mid - 1;
            else low = mid + 1;
        } else {                                // right half sorted
            if (a[mid] < target && target <= a[high]) low = mid + 1;
            else high = mid - 1;
        }
    }
    return -1;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> The key trick: determine the sorted half first, then check if the target falls within its known range.
