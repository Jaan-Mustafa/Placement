# Median of Two Sorted Arrays  🔴

**Reference:** [LeetCode 4 — Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

---

## Problem
Given two sorted arrays, find the median of their combined sorted form in **O(log(min(n,m)))**.

```
Input:  a = [1, 3], b = [2]       Output: 2.0
Input:  a = [1, 2], b = [3, 4]    Output: 2.5
```

---

## Intuition: binary search on the partition
Partition both arrays so the left side holds exactly half the total elements. Binary search the cut in the **smaller** array (`a`); the cut in `b` is determined (`(n+m+1)/2 - cutA`). A valid partition satisfies:

`l1 = a[cutA-1] ≤ r2 = b[cutB]`  **and**  `l2 = b[cutB-1] ≤ r1 = a[cutA]`.

- If `l1 > r2` → cut in `a` is too far right, move left.
- If `l2 > r1` → too far left, move right.

Use ±∞ sentinels for out-of-range edges.

---

## Code (C++)
```cpp
double findMedianSortedArrays(vector<int>& a, vector<int>& b) {
    if (a.size() > b.size()) return findMedianSortedArrays(b, a);  // search smaller
    int n = a.size(), m = b.size();
    int total = n + m, half = (total + 1) / 2;
    int low = 0, high = n;
    while (low <= high) {
        int cutA = low + (high - low) / 2;
        int cutB = half - cutA;

        int l1 = (cutA == 0) ? INT_MIN : a[cutA - 1];
        int r1 = (cutA == n) ? INT_MAX : a[cutA];
        int l2 = (cutB == 0) ? INT_MIN : b[cutB - 1];
        int r2 = (cutB == m) ? INT_MAX : b[cutB];

        if (l1 <= r2 && l2 <= r1) {                 // valid partition
            if (total % 2) return max(l1, l2);      // odd: max of left side
            return (max(l1, l2) + min(r1, r2)) / 2.0;
        } else if (l1 > r2) {
            high = cutA - 1;                        // move cut in a left
        } else {
            low = cutA + 1;                         // move cut in a right
        }
    }
    return 0.0;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log(min(n, m))) |
| Space | O(1) |

> Searching the **smaller** array keeps `cutB` in range and bounds the log factor. The ±∞ sentinels elegantly handle the empty-left / empty-right edge cases.
