# Grid Unique Paths  🟡

**Reference:** https://leetcode.com/problems/unique-paths/

---

## Problem

A robot starts at the top-left cell `(0,0)` of an `m x n` grid and wants to reach the bottom-right cell `(m-1, n-1)`. It can move only **right** or **down** at each step. Count the number of distinct paths.

```
m = 3, n = 2 grid:

  S . 
  . . 
  . F

Paths: D D R, D R D, R D D  -> 3 unique paths
```

---

## Intuition

State: `dp[i][j]` = number of unique paths from `(0,0)` to `(i,j)`.

To reach `(i,j)` the robot's last move came either from the left `(i,j-1)` or from above `(i-1,j)`:

```
dp[i][j] = dp[i-1][j] + dp[i][j-1]
```

Base case: the entire first row and first column have exactly one path (keep going right / keep going down), so `dp[0][*] = dp[*][0] = 1`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int uniquePaths(int m, int n) {
    // dp[i][j] = number of unique paths from (0,0) to (i,j)
    vector<vector<int>> dp(m, vector<int>(n, 0));

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0 && j == 0) {
                dp[i][j] = 1;                 // base: single way to stand at start
            } else {
                int up   = (i > 0) ? dp[i-1][j] : 0;   // came from above
                int left = (j > 0) ? dp[i][j-1] : 0;   // came from left
                dp[i][j] = up + left;
            }
        }
    }
    return dp[m-1][n-1];
}

int main() {
    cout << uniquePaths(3, 2) << "\n"; // 3
    cout << uniquePaths(3, 7) << "\n"; // 28
}
```

Space optimization: each row depends only on the previous row, so you can keep a single 1D `vector<int> prev(n, 1)` and update in place: `cur[j] = cur[j-1] + prev[j]`. That reduces space to O(n). (There is also a pure combinatorics answer: C(m+n-2, m-1).)

---

## Complexity

| | |
|---|---|
| Time | O(m × n) |
| Space | O(m × n), optimizable to O(n) |

> ⚠️ Path counts grow large; for big grids the value can overflow 32-bit `int`. LeetCode's constraints keep the answer within `int`, but if you generalize, use `long long`.
