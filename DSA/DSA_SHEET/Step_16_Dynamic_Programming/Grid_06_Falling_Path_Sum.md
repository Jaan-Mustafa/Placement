# Minimum / Maximum Falling Path Sum  🟡

**Reference:** https://leetcode.com/problems/minimum-falling-path-sum/

---

## Problem

Given an `n x n` matrix, a *falling path* starts at any cell in the **first row** and chooses one cell per row, moving down to a column that differs by at most one (`j-1`, `j`, or `j+1`). Return the **minimum** sum of a falling path. (For the maximum variant, swap `min` for `max`.)

```
matrix =
  [[2, 1, 3],
   [6, 5, 4],
   [7, 8, 9]]

Best falling path: 1 (row0,col1) -> 5 (row1,col1) -> 7 (row2,col0) = 13
Minimum falling path sum = 13
```

---

## Intuition

State: `dp[i][j]` = minimum sum of a falling path that **starts at the top row and ends at cell `(i,j)`**.

The step into `(i,j)` came from the cell directly above or one of the two diagonals above:

```
dp[i][j] = matrix[i][j] + min( dp[i-1][j-1], dp[i-1][j], dp[i-1][j+1] )
```

(out-of-range diagonals are treated as +∞). Base case: `dp[0][j] = matrix[0][j]`. The answer is the minimum over the entire **last row**, since a path may end at any bottom column.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int minFallingPathSum(vector<vector<int>>& matrix) {
    int n = matrix.size();
    // prev[j] = min falling-path sum ending at row i-1, column j.
    vector<int> prev(matrix[0]);          // base case: first row

    for (int i = 1; i < n; i++) {
        vector<int> cur(n);
        for (int j = 0; j < n; j++) {
            int up        = prev[j];                                  // straight down
            int leftDiag  = (j > 0)     ? prev[j-1] : INT_MAX;        // from upper-left
            int rightDiag = (j < n - 1) ? prev[j+1] : INT_MAX;        // from upper-right
            cur[j] = matrix[i][j] + min({up, leftDiag, rightDiag});
        }
        prev = cur;                       // roll forward one row
    }

    // A valid path may end at any column of the last row.
    return *min_element(prev.begin(), prev.end());
}

int main() {
    vector<vector<int>> m = {{2,1,3},{6,5,4},{7,8,9}};
    cout << minFallingPathSum(m) << "\n"; // 13
}
```

Maximum variant: replace `min({...})` with `max({...})`, use `-INF` sentinels for out-of-range diagonals, and return `*max_element(...)`.

Space optimization: already applied — only two rows (`prev`, `cur`) of size `n` are kept instead of the full `n x n` table, giving O(n) space.

---

## Complexity

| | |
|---|---|
| Time | O(n²) (each cell does O(1) work) |
| Space | O(n) (two rolling rows) |

> ⚠️ The path can **start and end at any column** — that is the key difference from grid min-path-sum. Take the min/max over the whole last row, not just `dp[n-1][0]`. Guard out-of-range diagonals with `INT_MAX` (or `-INT_MAX` for the max version) so border cells don't read invalid neighbors.
