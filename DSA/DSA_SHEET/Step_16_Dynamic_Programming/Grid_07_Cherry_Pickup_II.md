# Cherry Pickup II (3D DP)  🔴

**Reference:** https://leetcode.com/problems/cherry-pickup-ii/

---

## Problem

Given a `rows x cols` grid where `grid[r][c]` is the number of cherries in that cell, **two robots** start at the top row — robot 1 at `(0,0)` and robot 2 at `(0,cols-1)`. Both move down to the last row simultaneously. From `(r,c)` each robot may move to `(r+1,c-1)`, `(r+1,c)`, or `(r+1,c+1)`. When a robot passes a cell it collects all its cherries; if both robots are on the **same cell** the cherries are counted **only once**. Maximize the total cherries collected.

```
grid =
  [[3, 1, 1],
   [2, 5, 1],
   [1, 5, 5],
   [2, 1, 1]]

Two optimal descents collect 24 cherries total.
Answer = 24
```

---

## Intuition

Both robots are always on the **same row** `r` (they move in lockstep), so the state is the row plus each robot's column:

State: `dp[r][c1][c2]` = max cherries both robots collect travelling from row `r` (robot1 at `c1`, robot2 at `c2`) down to the last row.

Reward at a step: `grid[r][c1] + (c1 == c2 ? 0 : grid[r][c2])` — avoid double-counting when they overlap.

Transition: each robot independently picks one of 3 columns, so 9 combinations:

```
dp[r][c1][c2] = reward + max over dc1,dc2 in {-1,0,+1} ( dp[r+1][c1+dc1][c2+dc2] )
```

Base case (last row): `dp[rows-1][c1][c2] = grid[r][c1] + (c1==c2 ? 0 : grid[r][c2])`. Answer is `dp[0][0][cols-1]`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int cherryPickup(vector<vector<int>>& grid) {
    int rows = grid.size(), cols = grid[0].size();
    const int NEG = -1e9;

    // dp[c1][c2] for the row currently being processed (we roll over rows).
    // Initialize with the LAST row (base case).
    vector<vector<int>> dp(cols, vector<int>(cols, 0));
    for (int c1 = 0; c1 < cols; c1++)
        for (int c2 = 0; c2 < cols; c2++)
            dp[c1][c2] = grid[rows-1][c1] + (c1 == c2 ? 0 : grid[rows-1][c2]);

    // Process rows from second-to-last up to row 0.
    for (int r = rows - 2; r >= 0; r--) {
        vector<vector<int>> cur(cols, vector<int>(cols, 0));
        for (int c1 = 0; c1 < cols; c1++) {
            for (int c2 = 0; c2 < cols; c2++) {
                int best = NEG;
                // Each robot tries -1 / 0 / +1 column shift.
                for (int dc1 = -1; dc1 <= 1; dc1++) {
                    for (int dc2 = -1; dc2 <= 1; dc2++) {
                        int nc1 = c1 + dc1, nc2 = c2 + dc2;
                        if (nc1 >= 0 && nc1 < cols && nc2 >= 0 && nc2 < cols)
                            best = max(best, dp[nc1][nc2]);   // from the row below
                    }
                }
                int reward = grid[r][c1] + (c1 == c2 ? 0 : grid[r][c2]);
                cur[c1][c2] = reward + best;
            }
        }
        dp = cur;                       // roll up one row
    }

    // Robots start at the two top corners.
    return dp[0][cols-1];
}

int main() {
    vector<vector<int>> grid = {{3,1,1},{2,5,1},{1,5,5},{2,1,1}};
    cout << cherryPickup(grid) << "\n"; // 24
}
```

Space optimization: already applied — instead of the full `rows x cols x cols` cube we keep only two `cols x cols` layers (`dp` and `cur`), giving O(cols²) space.

---

## Complexity

| | |
|---|---|
| Time | O(rows × cols² × 9) = O(rows × cols²) |
| Space | O(cols²) (two rolling layers; full 3D table is O(rows × cols²)) |

> ⚠️ When `c1 == c2` the robots share a cell — add `grid[r][c1]` **only once**, otherwise you double-count and overshoot. Both robots always sit on the same row, which is exactly why the state needs only `(row, c1, c2)` and not two independent rows.
