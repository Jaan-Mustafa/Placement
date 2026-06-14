# Count Square Submatrices with All 1's  🟡

**Reference:** https://leetcode.com/problems/count-square-submatrices-with-all-ones/

---

## Problem

Given an `m x n` binary matrix, return the **total number of square submatrices** that contain only `1`s (counting squares of every size: 1x1, 2x2, ...).

```
matrix =
0 1 1 1
1 1 1 1
0 1 1 1
Total squares of all 1s = 15
```

---

## Intuition

DP on squares. Define `dp[i][j]` = the side length of the **largest all-1 square whose bottom-right corner is `(i, j)`**.

Recurrence:
- If `matrix[i][j] == 0`: `dp[i][j] = 0`.
- Else: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`.

Why `min`: a square of side `s` ending at `(i,j)` requires squares of side `s-1` ending at the three neighbours (top, left, top-left). The smallest of them limits how large this corner's square can be.

Key counting insight: a cell with `dp[i][j] = s` is simultaneously the bottom-right corner of exactly `s` squares (sizes `1..s`). Therefore the **answer is the sum of all `dp[i][j]`**.

Base cases: first row / first column just copy the cell value (a 1 there is a 1x1 square).

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int countSquares(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    // dp[i][j]: side of largest all-1 square with bottom-right at (i,j)
    vector<vector<int>> dp(m, vector<int>(n, 0));
    int total = 0;

    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (matrix[i][j] == 0) {
                dp[i][j] = 0;
            } else if (i == 0 || j == 0) {
                dp[i][j] = 1;                       // border cell: only 1x1
            } else {
                dp[i][j] = 1 + min({ dp[i - 1][j],
                                     dp[i][j - 1],
                                     dp[i - 1][j - 1] });
            }
            total += dp[i][j];                      // each cell adds its dp value
        }
    }
    return total;
}
```

> Space optimization: each row depends only on the previous row, so you can keep a single `prev`/`curr` row (O(n)) — or even update `matrix` in place if mutation is allowed.

---

## Complexity

| | |
|---|---|
| Time | O(m·n) |
| Space | O(m·n), reducible to O(n) |

> ⚠️ The answer is the **sum** of all `dp` values, not `max`. A cell with value `s` represents `s` distinct squares (one of each size up to `s`), so summing counts every square exactly once.
