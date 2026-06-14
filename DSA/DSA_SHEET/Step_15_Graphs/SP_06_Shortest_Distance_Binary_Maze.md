# Shortest Distance in a Binary Maze  🟡

**Reference:** [LeetCode 1091 — Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/)

---

## Problem
Given an `n x n` grid of `0`s (open) and `1`s (blocked), find the length of the shortest **clear path** from top-left `(0,0)` to bottom-right `(n-1,n-1)`. You may move in all **8 directions**; path length counts visited cells. Return `-1` if no path exists.

```
Input: grid = [[0,0,0],[1,1,0],[1,1,0]]
Output: 4   (cells (0,0)->(0,1)->(0,2)->(1,2)->(2,2))
```

---

## Intuition
Every move costs exactly one cell, so the grid behaves like an unweighted graph and plain **BFS** finds the shortest path. We start from `(0,0)` (if it is open), expand in all 8 directions, and mark cells visited as we enqueue them. The first time BFS reaches `(n-1,n-1)` we return the accumulated path length.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
    int n = grid.size();
    if (grid[0][0] == 1 || grid[n-1][n-1] == 1) return -1;   // blocked ends

    // 8 directions
    int dr[8] = {-1,-1,-1, 0, 0, 1, 1, 1};
    int dc[8] = {-1, 0, 1,-1, 1,-1, 0, 1};

    queue<tuple<int,int,int>> q;          // {row, col, pathLength}
    q.push({0, 0, 1});
    grid[0][0] = 1;                       // mark visited (reuse grid)

    while (!q.empty()) {
        auto [r, c, dist] = q.front(); q.pop();
        if (r == n-1 && c == n-1) return dist;   // reached destination
        for (int k = 0; k < 8; ++k) {
            int nr = r + dr[k], nc = c + dc[k];
            if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 0) {
                grid[nr][nc] = 1;          // mark before pushing (avoid dupes)
                q.push({nr, nc, dist + 1});
            }
        }
    }
    return -1;                            // destination unreachable
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n²) |
| Space | O(n²) |

> Mark a cell visited at enqueue time, not dequeue time — otherwise the same cell gets pushed many times and BFS degrades badly.
