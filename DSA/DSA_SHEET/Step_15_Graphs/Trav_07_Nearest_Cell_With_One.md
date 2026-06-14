# Distance of Nearest Cell Having 1 (0/1 Matrix)  🟡

**Reference:** [LeetCode 542 — 01 Matrix](https://leetcode.com/problems/01-matrix/)

---

## Problem
Given a binary grid, for every cell compute the distance to the nearest cell containing `1`, where distance is the number of 4-directional steps.

> Note: LeetCode 542 asks for distance to the nearest `0`. The classic GFG framing "distance of nearest cell having 1" is the mirror image — seed BFS from all `1`s instead. The code below solves the LeetCode version (nearest `0`); to get "nearest 1", just swap which value seeds the queue.

```
Input: mat = [[0,0,0],[0,1,0],[1,1,1]]
Output: [[0,0,0],[0,1,0],[1,1,1]] -> dist to nearest 0:
        [[0,0,0],[0,1,0],[1,2,1]]
```

---

## Intuition
Single-source BFS would be slow if run from each cell. Instead use multi-source BFS: seed the queue with **all** the target cells (distance 0) at once and expand outward. Because BFS explores in increasing distance order, the first time we reach any cell we have found its shortest distance. A `dist` matrix doubles as the visited marker (`-1` = unvisited).

---

## Code (C++)
```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();
        vector<vector<int>> dist(n, vector<int>(m, -1));    // -1 = not yet reached
        queue<pair<int,int>> q;

        // seed every 0 as a BFS source (swap to mat[i][j]==1 for "nearest 1")
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (mat[i][j] == 0) { dist[i][j] = 0; q.push({i, j}); }

        int dr[] = {-1, 1, 0, 0}, dc[] = {0, 0, -1, 1};
        while (!q.empty()) {
            auto [r, c] = q.front(); q.pop();
            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m && dist[nr][nc] == -1) {
                    dist[nr][nc] = dist[r][c] + 1;          // one step farther
                    q.push({nr, nc});
                }
            }
        }
        return dist;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N*M) |
| Space | O(N*M) |

> Seeding all sources at level 0 is the whole trick — it turns N*M separate BFS runs into a single one while still giving each cell its true minimum.
