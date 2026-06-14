# Median in a Row-Wise Sorted Matrix  🔴

**Reference:** [GeeksforGeeks — Median in a row-wise sorted matrix](https://www.geeksforgeeks.org/median-of-row-wise-sorted-matrix/)

---

## Problem
Given an `m × n` matrix (m·n is odd) where every row is sorted, find the **median** of all elements — without flattening and sorting (which is O(m·n·log(m·n))).

```
Input:  [[1, 3, 5],
         [2, 6, 9],
         [3, 6, 9]]
Output: 5
```

---

## Intuition: binary search on the value
The median is the value with exactly `(m*n)/2` elements smaller than it. Binary search on the **value range** `[min element, max element]`:
- For a candidate `mid`, count how many elements are ≤ `mid` (sum of `upperBound` over each sorted row).
- If that count ≤ `(m*n)/2`, the median is larger → go right; else go left.

We never materialize the merged array.

---

## Code (C++)
```cpp
int countLessEqual(vector<vector<int>>& mat, int x) {
    int count = 0;
    for (auto& row : mat)
        count += upper_bound(row.begin(), row.end(), x) - row.begin();
    return count;
}

int matrixMedian(vector<vector<int>>& mat) {
    int n = mat.size(), m = mat[0].size();
    int low = INT_MAX, high = INT_MIN;
    for (auto& row : mat) {
        low  = min(low, row[0]);            // smallest element
        high = max(high, row[m - 1]);       // largest element
    }
    int need = (n * m) / 2;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (countLessEqual(mat, mid) <= need) low = mid + 1;
        else high = mid - 1;
    }
    return low;                             // median value
}
```

---

## Why `low` is the answer
We seek the smallest value whose count-of-≤ exceeds `(m*n)/2`. The binary search converges with `low` landing on exactly that value — the median.

## Complexity
| | |
|---|---|
| Time | O(log(maxVal − minVal) · m · log n) |
| Space | O(1) |

> Binary-searching on the **answer value** (not indices) is the key idea — the same trick used for "kth smallest in a sorted matrix".
