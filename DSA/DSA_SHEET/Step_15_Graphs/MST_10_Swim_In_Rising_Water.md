# Swim in Rising Water  🔴

**Reference:** [LeetCode 778 — Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/)

---

## Problem
On an `n x n` grid, `grid[r][c]` is the elevation. At time `t`, all cells with elevation `<= t` are submerged and swimmable. Starting at `(0,0)`, find the least time `t` to reach `(n-1, n-1)`, moving 4-directionally.

```
Input: grid=[[0,2],[1,3]]
Output: 3   (must wait for cell (1,1)=3 to be reachable)
```

---

## Intuition
We want the path from top-left to bottom-right that **minimizes the maximum elevation** encountered (the bottleneck). This is a minimax-path problem. Use a Dijkstra-style min-heap keyed by "the highest elevation needed so far to reach this cell". Always expand the cell with the smallest such bottleneck. When a neighbor is reached, its cost is `max(currentBottleneck, neighborElevation)`. The first time we pop the destination, that bottleneck is the answer — because any other route would have an equal-or-greater peak.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int swimInWater(vector<vector<int>>& grid) {
    int n = grid.size();
    // min-heap of {bottleneck-so-far, row, col}
    priority_queue<array<int,3>, vector<array<int,3>>, greater<>> pq;
    vector<vector<bool>> visited(n, vector<bool>(n, false));
    int dr[] = {-1, 1, 0, 0}, dc[] = {0, 0, -1, 1};

    pq.push({grid[0][0], 0, 0});                      // start cell's own height
    while (!pq.empty()) {
        auto [t, r, c] = pq.top(); pq.pop();
        if (visited[r][c]) continue;                  // stale heavier entry
        visited[r][c] = true;
        if (r == n - 1 && c == n - 1) return t;       // reached goal: minimax done

        for (int d = 0; d < 4; ++d) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nr >= n || nc < 0 || nc >= n || visited[nr][nc]) continue;
            // cost to stand on neighbour = max of path peak and its elevation
            pq.push({max(t, grid[nr][nc]), nr, nc});
        }
    }
    return -1;                                        // unreachable (won't happen on a full grid)
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n² log n) |
| Space | O(n²) |

> The heap key is the path's running **maximum**, not a running sum — that single change turns Dijkstra into a minimax (widest/bottleneck-path) solver; a DSU + binary-search-on-time variant also works.
