# Floyd-Warshall Algorithm  🔴

**Reference:** [GeeksforGeeks — Floyd-Warshall Algorithm](https://www.geeksforgeeks.org/floyd-warshall-algorithm-dp-16/)

---

## Problem
Given an `n x n` adjacency matrix `dist` where `dist[i][j]` is the edge weight (`-1` / a large value if no edge), compute the shortest distance between **every pair** of nodes in place. Works with negative edges (no negative cycles).

```
Input: dist = [[0,  1, 43],
               [1,  0,  6],
               [-1, -1, 0]]   (-1 means no edge)
Output: [[0, 1, 7],
         [1, 0, 6],
         [INF, INF, 0]]
```

---

## Intuition
Floyd-Warshall is dynamic programming over an intermediate vertex `k`: the shortest path from `i` to `j` either avoids `k` or routes through it. The triple loop with `k` outermost progressively allows more intermediate vertices, so after all `k` the matrix holds all-pairs shortest paths. We first convert "no edge" sentinels to a large value to make `min` arithmetic clean.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

void floydWarshall(vector<vector<int>>& dist) {
    int n = dist.size();
    const int INF = 1e9;

    // convert "no edge" (-1) to INF
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            if (dist[i][j] == -1) dist[i][j] = INF;

    // try every node k as an intermediate vertex
    for (int k = 0; k < n; ++k)
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                if (dist[i][k] < INF && dist[k][j] < INF)
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);

    // negative cycle check: dist[i][i] < 0 means one exists
    // for (int i = 0; i < n; ++i) if (dist[i][i] < 0) { /* negative cycle */ }

    // convert INF back to -1 for "unreachable"
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            if (dist[i][j] == INF) dist[i][j] = -1;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n³) |
| Space | O(1) extra (in place) |

> The loop order matters: `k` must be the **outermost** loop, and a negative value on the diagonal (`dist[i][i] < 0`) signals a negative cycle.
