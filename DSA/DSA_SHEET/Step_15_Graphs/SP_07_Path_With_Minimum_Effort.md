# Path With Minimum Effort  🟡

**Reference:** [LeetCode 1631 — Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/)

---

## Problem
Given a `rows x cols` grid of heights, travel from top-left to bottom-right moving in 4 directions. A path's **effort** is the maximum absolute height difference between consecutive cells along it. Return the minimum possible effort.

```
Input: heights = [[1,2,2],[3,8,2],[5,3,5]]
Output: 2   (path 1->2->2->2->5, max step diff = 2)
```

---

## Intuition
Replace "sum of edge weights" with "max edge weight along the path" and Dijkstra still works. We keep `effort[r][c]` = the minimum possible *maximum step* to reach that cell, and a min-heap ordered by that running maximum. When relaxing a neighbour, the new effort is `max(currentEffort, |heightDiff|)`; if that beats the stored value we push it. The first time we pop the destination, its effort is final.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int minimumEffortPath(vector<vector<int>>& heights) {
    int n = heights.size(), m = heights[0].size();
    vector<vector<int>> effort(n, vector<int>(m, INT_MAX));

    // min-heap of {effortSoFar, row, col}
    priority_queue<tuple<int,int,int>,
                   vector<tuple<int,int,int>>, greater<>> pq;
    effort[0][0] = 0;
    pq.push({0, 0, 0});

    int dr[4] = {-1, 1, 0, 0}, dc[4] = {0, 0, -1, 1};

    while (!pq.empty()) {
        auto [e, r, c] = pq.top(); pq.pop();
        if (r == n-1 && c == m-1) return e;       // destination finalised
        if (e > effort[r][c]) continue;           // stale entry
        for (int k = 0; k < 4; ++k) {
            int nr = r + dr[k], nc = c + dc[k];
            if (nr >= 0 && nr < n && nc >= 0 && nc < m) {
                // effort = max of current path effort and this step's diff
                int ne = max(e, abs(heights[nr][nc] - heights[r][c]));
                if (ne < effort[nr][nc]) {
                    effort[nr][nc] = ne;
                    pq.push({ne, nr, nc});
                }
            }
        }
    }
    return 0;                                     // single-cell grid
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n·m·log(n·m)) |
| Space | O(n·m) |

> The trick is `max` instead of `+` when relaxing — Dijkstra's correctness holds for any monotone path-cost function, not just sums.
