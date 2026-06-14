# Find Peak Element in 2D  🔴

**Reference:** [LeetCode 1901 — Find a Peak Element II](https://leetcode.com/problems/find-a-peak-element-ii/)

---

## Problem
In an `m × n` grid, a peak is a cell strictly greater than its 4 neighbors (up/down/left/right). Cells outside the grid are -∞. Return the coordinates of **any** peak in O(m log n) or O(n log m).

```
Input:  [[1, 4],
         [3, 2]]
Output: [0, 1]   (value 4 is greater than its neighbors)
```

---

## Intuition: binary search on columns
Binary search the **column** index. For the middle column, find the row of its **maximum** element. Compare that max with its left/right neighbors:
- If left neighbor is bigger → a peak exists in the left half of columns.
- If right neighbor is bigger → a peak is in the right half.
- Otherwise this cell is a peak (it's already the column max, so up/down are smaller).

The column's max guarantees the vertical neighbors are smaller, so we only check horizontally.

---

## Code (C++)
```cpp
int maxRowInCol(vector<vector<int>>& mat, int col) {
    int row = 0;
    for (int i = 0; i < mat.size(); i++)
        if (mat[i][col] > mat[row][col]) row = i;
    return row;
}

vector<int> findPeakGrid(vector<vector<int>>& mat) {
    int n = mat[0].size();
    int low = 0, high = n - 1;
    while (low <= high) {
        int midCol = low + (high - low) / 2;
        int row = maxRowInCol(mat, midCol);
        int left  = (midCol - 1 >= 0) ? mat[row][midCol - 1] : -1;
        int right = (midCol + 1 < n)  ? mat[row][midCol + 1] : -1;

        if (mat[row][midCol] > left && mat[row][midCol] > right)
            return {row, midCol};               // peak found
        else if (left > mat[row][midCol])
            high = midCol - 1;                  // peak to the left
        else
            low = midCol + 1;                   // peak to the right
    }
    return {-1, -1};
}
```

---

## Complexity
| | |
|---|---|
| Time | O(m · log n) — log n columns, each O(m) to find the column max |
| Space | O(1) |

> The trick: picking the column's maximum cell guarantees its up/down neighbors are smaller, reducing the 2D problem to a 1D "find peak" on the chosen column.
