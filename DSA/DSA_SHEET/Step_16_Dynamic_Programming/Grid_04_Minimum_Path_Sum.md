# Minimum Path Sum in Grid  🟡

**Reference:** https://leetcode.com/problems/minimum-path-sum/

---

## Problem

Given an `m x n` grid of non-negative numbers, find a path from the top-left `(0,0)` to the bottom-right `(m-1,n-1)` that **minimizes the sum** of numbers along the path. You may move only right or down.

```
grid =
  [[1, 3, 1],
   [1, 5, 1],
   [4, 2, 1]]

Best path: 1 -> 3 -> 1 -> 1 -> 1 = 7
Answer = 7
```

---

## Intuition

State: `dp[i][j]` = minimum cost to travel from `(0,0)` to `(i,j)`.

The last step into `(i,j)` arrives from above or from the left, so take the cheaper predecessor and add the current cell's value:

```
dp[i][j] = grid[i][j] + min( dp[i-1][j], dp[i][j-1] )
```

Base case: `dp[0][0] = grid[0][0]`. For the first row only `left` exists, for the first column only `up` exists.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    vector<vector<int>> dp(m, vector<int>(n, 0));

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0 && j == 0) {
                dp[i][j] = grid[i][j];        // base: start cell
            } else {
                // INT_MAX acts as "no predecessor" on the borders.
                int up   = (i > 0) ? dp[i-1][j] : INT_MAX;
                int left = (j > 0) ? dp[i][j-1] : INT_MAX;
                dp[i][j] = grid[i][j] + min(up, left);
            }
        }
    }
    return dp[m-1][n-1];
}

int main() {
    vector<vector<int>> grid = {{1,3,1},{1,5,1},{4,2,1}};
    cout << minPathSum(grid) << "\n"; // 7
}
```

Space optimization: each row only needs the row above, so a single 1D `vector<int> prev(n)` rolled forward gives O(n) space.

---

## Complexity

| | |
|---|---|
| Time | O(m × n) |
| Space | O(m × n), optimizable to O(n) |

> ⚠️ Using `INT_MAX` as the sentinel for a missing border predecessor is safe here only because we always also pass through a valid neighbor at non-start cells. Never add `grid[i][j]` to `INT_MAX` directly — guard with `min()` first as shown, otherwise you get integer overflow.
