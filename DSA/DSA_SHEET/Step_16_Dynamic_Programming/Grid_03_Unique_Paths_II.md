# Grid Unique Paths II (with Obstacles)  🟡

**Reference:** https://leetcode.com/problems/unique-paths-ii/

---

## Problem

Same as Unique Paths, but the `m x n` grid contains **obstacles** marked with `1` (free cells are `0`). The robot moves only right or down and cannot step on an obstacle. Count distinct paths from top-left to bottom-right.

```
grid =
  [[0, 0, 0],
   [0, 1, 0],
   [0, 0, 0]]

The single obstacle at (1,1) blocks paths through the center.
Answer = 2  (around the top or around the bottom)
```

---

## Intuition

State: `dp[i][j]` = number of unique paths from `(0,0)` to `(i,j)` avoiding obstacles.

Recurrence — if the cell itself is an obstacle, no path can stand on it, so `dp[i][j] = 0`; otherwise it's the standard sum:

```
dp[i][j] = 0                                if grid[i][j] == 1
dp[i][j] = dp[i-1][j] + dp[i][j-1]          otherwise
```

Base case: `dp[0][0] = (grid[0][0] == 0) ? 1 : 0`. Note the start or end cell could itself be blocked.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int uniquePathsWithObstacles(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    vector<vector<int>> dp(m, vector<int>(n, 0));

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 1) {
                dp[i][j] = 0;                 // obstacle: unreachable, contributes nothing
            } else if (i == 0 && j == 0) {
                dp[i][j] = 1;                 // start cell (already known to be free here)
            } else {
                int up   = (i > 0) ? dp[i-1][j] : 0;
                int left = (j > 0) ? dp[i][j-1] : 0;
                dp[i][j] = up + left;
            }
        }
    }
    return dp[m-1][n-1];
}

int main() {
    vector<vector<int>> grid = {{0,0,0},{0,1,0},{0,0,0}};
    cout << uniquePathsWithObstacles(grid) << "\n"; // 2
}
```

Space optimization: as in the no-obstacle version, only the previous row is needed, so a single 1D array of size `n` suffices — set the slot to 0 when the current cell is an obstacle. This brings space down to O(n).

---

## Complexity

| | |
|---|---|
| Time | O(m × n) |
| Space | O(m × n), optimizable to O(n) |

> ⚠️ Always check `grid[0][0]` (and the destination) for an obstacle first — if the start is blocked the answer is 0. Use `long long` if path counts can exceed 32-bit range for larger generalized inputs.
