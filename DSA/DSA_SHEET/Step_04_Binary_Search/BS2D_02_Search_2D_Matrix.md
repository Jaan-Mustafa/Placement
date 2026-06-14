# Search in a 2D Matrix  🟡

**Reference:** [LeetCode 74 — Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)

---

## Problem
In an `m × n` matrix where each row is sorted **and** the first element of each row is greater than the last of the previous row (fully sorted if flattened), determine if `target` exists.

```
Input:  [[1, 3, 5, 7],
         [10, 11, 16, 20],
         [23, 30, 34, 60]], target = 16
Output: true
```

---

## Intuition: treat it as a flat sorted array
Because the matrix is fully sorted when read row by row, we can binary search over the virtual 1D array of length `m*n`. Map a flat index `mid` to 2D coordinates: `row = mid / cols`, `col = mid % cols`.

---

## Code (C++)
```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int low = 0, high = m * n - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        int val = matrix[mid / n][mid % n];     // 1D index → 2D
        if (val == target) return true;
        else if (val < target) low = mid + 1;
        else high = mid - 1;
    }
    return false;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log(m · n)) |
| Space | O(1) |

> ⚠️ The index mapping uses the **column count** `n`: `row = mid / n`, `col = mid % n`. This works only because the whole matrix is one sorted sequence.
