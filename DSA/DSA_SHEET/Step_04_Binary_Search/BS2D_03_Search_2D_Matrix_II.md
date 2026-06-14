# Search in a 2D Matrix II  🟡

**Reference:** [LeetCode 240 — Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/)

---

## Problem
Each row is sorted left→right and each column is sorted top→bottom, but rows are **not** globally chained (the first of a row is not necessarily larger than the previous row's last). Search for `target`.

```
Input:  [[1, 4, 7, 11],
         [2, 5, 8, 12],
         [3, 6, 9, 16],
         [10, 13, 14, 17]], target = 5
Output: true
```

---

## Intuition: staircase search from a corner
Start at the **top-right** corner. That element is the largest in its row and smallest in its column, so each comparison rules out a full row or column:
- If it equals target → found.
- If it's **greater** than target → this column can't contain the target below → move **left**.
- If it's **less** than target → this row is exhausted → move **down**.

---

## Code (C++)
```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int row = 0, col = n - 1;                  // start top-right
    while (row < m && col >= 0) {
        int val = matrix[row][col];
        if (val == target) return true;
        else if (val > target) col--;          // eliminate this column
        else row++;                            // eliminate this row
    }
    return false;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(m + n) |
| Space | O(1) |

> You could also binary search each row in O(m log n), but the staircase is simpler and optimal here. Top-left or bottom-right corners don't work — they don't give a clear "eliminate row vs column" decision.
