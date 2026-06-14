# Set Matrix Zeroes  🟡

**Reference:** [LeetCode 73 — Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)

---

## Problem
If an element in an `m × n` matrix is 0, set its entire row and column to 0. Do it in place.

```
Input:  [[1,1,1],[1,0,1],[1,1,1]]
Output: [[1,0,1],[0,0,0],[1,0,1]]
```

---

## Intuition
We must record which rows/cols to zero **before** overwriting (else cascading zeros corrupt the data). The O(1)-space trick uses the **first row and first column as markers**:
- For each zero at `(i,j)`, mark `matrix[i][0] = 0` and `matrix[0][j] = 0`.
- A separate flag handles column 0 itself (since `matrix[0][0]` is shared between row-0 and col-0 markers).
- Then zero cells based on the markers, processing the first row/col last.

---

## Code (C++)
```cpp
void setZeroes(vector<vector<int>>& m) {
    int rows = m.size(), cols = m[0].size();
    bool col0 = false;                       // does column 0 need zeroing?

    for (int i = 0; i < rows; i++) {
        if (m[i][0] == 0) col0 = true;
        for (int j = 1; j < cols; j++)
            if (m[i][j] == 0) {              // mark in first row & col
                m[i][0] = 0;
                m[0][j] = 0;
            }
    }
    // fill from bottom-right so markers aren't overwritten early
    for (int i = rows - 1; i >= 0; i--) {
        for (int j = cols - 1; j >= 1; j--)
            if (m[i][0] == 0 || m[0][j] == 0) m[i][j] = 0;
        if (col0) m[i][0] = 0;
    }
}
```

---

## Complexity
| | |
|---|---|
| Time | O(m × n) |
| Space | O(1) (markers stored in the matrix itself) |

> Simpler O(m+n) space version: keep two boolean arrays `row[]` and `col[]` of which rows/cols contain a zero.
