# Rotten Oranges  🟡

**Reference:** [LeetCode 994 — Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

---

## Problem
In a grid, `0` = empty, `1` = fresh orange, `2` = rotten orange. Every minute, a rotten orange rots all fresh oranges in its 4 adjacent cells. Return the minimum minutes until no fresh orange remains, or `-1` if impossible.

```
Input: grid = [[2,1,1],[1,1,0],[0,1,1]]
Output: 4
```

---

## Intuition
This is a multi-source BFS. All initially rotten oranges spread simultaneously, so we seed the queue with every rotten cell at time 0 and process the grid level by level. Each BFS level corresponds to one minute. We count fresh oranges up front; every time we rot a fresh one we decrement the counter. At the end, if any fresh orange is left unreachable, return `-1`; otherwise the number of elapsed levels is the answer.

---

## Code (C++)
```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        queue<pair<int,int>> q;                 // holds currently rotten cells
        int fresh = 0;

        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 2) q.push({i, j});
                else if (grid[i][j] == 1) fresh++;
            }

        if (fresh == 0) return 0;               // nothing to rot
        int minutes = 0;
        int dr[] = {-1, 1, 0, 0}, dc[] = {0, 0, -1, 1};

        while (!q.empty()) {
            int sz = q.size();
            bool rottedThisLevel = false;
            for (int k = 0; k < sz; k++) {      // process one minute's worth
                auto [r, c] = q.front(); q.pop();
                for (int d = 0; d < 4; d++) {
                    int nr = r + dr[d], nc = c + dc[d];
                    if (nr >= 0 && nr < n && nc >= 0 && nc < m && grid[nr][nc] == 1) {
                        grid[nr][nc] = 2;        // rot the fresh orange
                        fresh--;
                        q.push({nr, nc});
                        rottedThisLevel = true;
                    }
                }
            }
            if (rottedThisLevel) minutes++;      // only count minutes that did work
        }

        return fresh == 0 ? minutes : -1;        // leftover fresh -> impossible
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N*M) |
| Space | O(N*M) |

> Increment the minute counter only when a level actually rots something — otherwise an already-finished grid would over-count by one.
