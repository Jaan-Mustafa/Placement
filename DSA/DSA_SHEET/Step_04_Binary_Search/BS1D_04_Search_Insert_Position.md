# Search Insert Position  🟢

**Reference:** [LeetCode 35 — Search Insert Position](https://leetcode.com/problems/search-insert-position/)

---

## Problem
Given a sorted array of distinct integers and a target, return the index if found; otherwise the index where it would be inserted to keep the array sorted.

```
Input:  [1, 3, 5, 6], target = 5    Output: 2
Input:  [1, 3, 5, 6], target = 2    Output: 1
Input:  [1, 3, 5, 6], target = 7    Output: 4
```

---

## Intuition
This is exactly the **lower bound** of the target: the first index where `a[i] >= target`. If the target exists, lower bound lands on it; otherwise it lands on the correct insertion slot.

---

## Code (C++)
```cpp
int searchInsert(vector<int>& a, int target) {
    int low = 0, high = a.size() - 1, ans = a.size();
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] >= target) {
            ans = mid;
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> Recognizing a problem as "just lower/upper bound" is half the battle in binary search.
