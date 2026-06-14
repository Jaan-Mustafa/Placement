# Making a Large Island  🔴

**Reference:** [LeetCode 827 — Making a Large Island](https://leetcode.com/problems/making-a-large-island/)

---

## Problem
Given an `n x n` binary grid, you may change at most one `0` to `1`. Return the size of the largest island (4-directionally connected `1`s) achievable.

```
Input: grid=[[1,0],[0,1]]
Output: 3   (flip either 0 to join the two single-cell islands)
```

---

## Intuition
First label every existing island with a Disjoint Set (union by **size** so each root knows its island's size). Then for every `0` cell, imagine flipping it: it would bridge all the **distinct** island roots among its four neighbors. Sum those unique island sizes and add one for the flipped cell itself — that is the candidate largest island. Track the best over all `0`s. Edge case: if the grid is entirely land, no flip happens and the answer is the whole grid's size.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, size_;
    DSU(int n): parent(n), size_(n, 1) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }
    void unite(int a, int b) {                       // union by size
        a = find(a); b = find(b);
        if (a == b) return;
        if (size_[a] < size_[b]) swap(a, b);
        parent[b] = a;
        size_[a] += size_[b];
    }
};

int largestIsland(vector<vector<int>>& grid) {
    int n = grid.size();
    DSU dsu(n * n);
    int dr[] = {-1, 1, 0, 0}, dc[] = {0, 0, -1, 1};

    // 1) union all adjacent land cells into islands
    for (int r = 0; r < n; ++r)
        for (int c = 0; c < n; ++c)
            if (grid[r][c] == 1)
                for (int d = 0; d < 4; ++d) {
                    int nr = r + dr[d], nc = c + dc[d];
                    if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 1)
                        dsu.unite(r * n + c, nr * n + nc);
                }

    int best = 0;
    bool hasZero = false;
    // 2) try flipping each 0; sum sizes of distinct neighbor islands + 1
    for (int r = 0; r < n; ++r)
        for (int c = 0; c < n; ++c) {
            if (grid[r][c] != 0) continue;
            hasZero = true;
            unordered_set<int> roots;
            int total = 1;                           // the flipped cell itself
            for (int d = 0; d < 4; ++d) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 1) {
                    int root = dsu.find(nr * n + nc);
                    if (roots.insert(root).second)   // count each island once
                        total += dsu.size_[root];
                }
            }
            best = max(best, total);
        }

    if (!hasZero) return n * n;                       // all land, no flip needed
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n²·α) |
| Space | O(n²) |

> The `unordered_set` of roots is essential — without deduping, a `0` touching the same big island from two sides would count its size twice and overstate the result.
