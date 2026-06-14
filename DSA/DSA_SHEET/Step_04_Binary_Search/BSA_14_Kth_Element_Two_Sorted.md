# Kth Element of Two Sorted Arrays  🔴

**Reference:** [GeeksforGeeks — K-th element of two sorted arrays](https://www.geeksforgeeks.org/k-th-element-two-sorted-arrays/)

---

## Problem
Given two sorted arrays, find the `k`-th smallest element of their merged sorted form (1-indexed), in O(log(min(n,m))).

```
Input:  a = [2, 3, 6, 7, 9], b = [1, 4, 8, 10], k = 5    Output: 6
```

---

## Intuition: partition (same as median)
This is the median problem generalized to any `k`. Partition so the left side has exactly `k` elements split between the two arrays. Binary search the cut in the smaller array; the cut in the other is `k - cutA`. A valid partition has `l1 ≤ r2` and `l2 ≤ r1`; the answer is `max(l1, l2)` (the largest element on the left side).

---

## Code (C++)
```cpp
int kthElement(vector<int>& a, vector<int>& b, int k) {
    if (a.size() > b.size()) return kthElement(b, a, k);
    int n = a.size(), m = b.size();
    // cutA can range so that cutB stays in [0, m]
    int low = max(0, k - m), high = min(k, n);
    while (low <= high) {
        int cutA = low + (high - low) / 2;
        int cutB = k - cutA;

        int l1 = (cutA == 0) ? INT_MIN : a[cutA - 1];
        int r1 = (cutA == n) ? INT_MAX : a[cutA];
        int l2 = (cutB == 0) ? INT_MIN : b[cutB - 1];
        int r2 = (cutB == m) ? INT_MAX : b[cutB];

        if (l1 <= r2 && l2 <= r1) return max(l1, l2);  // kth = largest on left
        else if (l1 > r2) high = cutA - 1;
        else low = cutA + 1;
    }
    return -1;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log(min(n, m))) |
| Space | O(1) |

> The bounds `low = max(0, k-m)` and `high = min(k, n)` ensure `cutB = k - cutA` never goes out of `[0, m]`. The median problem is just this with `k = (n+m+1)/2`.
