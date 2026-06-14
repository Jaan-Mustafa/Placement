# Merge Two Sorted Arrays Without Extra Space  🔴

**Reference:** [LeetCode 88 — Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/) · [GeeksforGeeks](https://www.geeksforgeeks.org/merge-two-sorted-arrays-o1-extra-space/)

---

## Problem
Given two sorted arrays `a` (size n) and `b` (size m), merge them so `a` holds the smallest n elements and `b` holds the largest m, both sorted — using O(1) extra space.

```
Input:  a = [1, 4, 8, 10], b = [2, 3, 9]
Output: a = [1, 2, 3, 4],  b = [8, 9, 10]
```

---

## Intuition: Gap method (Shell-sort style)
Compare/swap elements a `gap` apart, where the gap spans across both arrays treated as one virtual array of length `n+m`. Start `gap = ceil((n+m)/2)`, halve each round until gap = 0. Each pass pushes larger elements rightward across the boundary. This achieves O((n+m)·log(n+m)) time with O(1) space.

---

## Code (C++)
```cpp
void mergeNoExtra(vector<int>& a, vector<int>& b) {
    int n = a.size(), m = b.size(), total = n + m;
    int gap = (total + 1) / 2;                       // ceil

    auto val = [&](int idx) -> int& {                // index across both arrays
        return idx < n ? a[idx] : b[idx - n];
    };

    while (gap > 0) {
        int i = 0, j = gap;
        while (j < total) {
            if (val(i) > val(j)) swap(val(i), val(j));
            i++; j++;
        }
        gap = (gap == 1) ? 0 : (gap + 1) / 2;
    }
}
```

---

## Complexity
| | |
|---|---|
| Time | O((n+m) · log(n+m)) |
| Space | O(1) |

> The simpler LeetCode 88 (where `a` has trailing space of size m) merges from the **back** with three pointers in O(n+m) time — that's the preferred answer when extra room exists.
