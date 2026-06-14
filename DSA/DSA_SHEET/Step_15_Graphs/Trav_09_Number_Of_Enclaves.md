# Number of Enclaves  🟡

**Reference:** [LeetCode 1020 — Number of Enclaves](https://leetcode.com/problems/number-of-enclaves/)

---

## Problem
Given a binary grid (`1` = land, `0` = sea), a land cell can "walk off" the grid if a sequence of 4-directional moves over land reaches the boundary. Return the number of land cells from which you **cannot** walk off the grid.

```
Input: grid = [[0,0,0,0],[1,0,1,0],[0,1,1,0],[0,0,0,0]]
Output: 3
```

---

## Intuition
Mirror of Surrounded Regions. Any land connected to the border can escape, so it is never an enclave. DFS from every boundary land cell and sink that connected land to `0` (mark as escapable). After flooding all border-connected land, the only `1`s left are fully enclosed — count them.

---

## Code (C++)
```cpp
class Solution {
public:
    void dfs(int r, int c, vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        if (r < 0 || r >= n || c < 0 || c >= m || grid[r][c] == 0) return;
        grid[r][c] = 0;                         // sink border-connected land
        dfs(r - 1, c, grid); dfs(r + 1, c, grid);
        dfs(r, c - 1, grid); dfs(r, c + 1, grid);
    }

    int numEnclaves(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();

        // flood land touching any border
        for (int i = 0; i < n; i++) {
            if (grid[i][0] == 1)     dfs(i, 0, grid);
            if (grid[i][m-1] == 1)   dfs(i, m-1, grid);
        }
        for (int j = 0; j < m; j++) {
            if (grid[0][j] == 1)     dfs(0, j, grid);
            if (grid[n-1][j] == 1)   dfs(n-1, j, grid);
        }

        // remaining land is enclosed
        int count = 0;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (grid[i][j] == 1) count++;
        return count;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N*M) |
| Space | O(N*M) |

> Same "remove the escapable, count the rest" strategy as Surrounded Regions — only the marker semantics differ.
