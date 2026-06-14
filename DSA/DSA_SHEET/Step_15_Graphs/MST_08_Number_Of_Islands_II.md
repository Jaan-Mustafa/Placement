# Number of Islands II  🔴

**Reference:** [LeetCode 305 — Number of Islands II](https://leetcode.com/problems/number-of-islands-ii/)

---

## Problem
On an `m x n` grid of water, you add land one cell at a time (given by `positions`). After each addition, report the current number of islands (4-directionally connected land groups).

```
Input: m=3, n=3, positions=[[0,0],[0,1],[1,2],[2,1]]
Output: [1, 1, 2, 3]
```

---

## Intuition
This is dynamic connectivity — perfect for a Disjoint Set. Maintain a running island count. When a cell becomes land: if it was already land, the count is unchanged; otherwise increment the count by one (a brand-new island) and then check its four neighbors. For each neighbor that is land and belongs to a different set, `unite` them and decrement the count (two islands merged into one). Flatten 2D coordinates to `r*n + c` as DSU indices.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    DSU(int n): parent(n), rank_(n, 0) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }
    bool unite(int a, int b) {                       // returns true if merged
        a = find(a); b = find(b);
        if (a == b) return false;
        if (rank_[a] < rank_[b]) swap(a, b);
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;
        return true;
    }
};

vector<int> numIslands2(int m, int n, vector<vector<int>>& positions) {
    DSU dsu(m * n);
    vector<bool> isLand(m * n, false);
    vector<int> res;
    int count = 0;
    int dr[] = {-1, 1, 0, 0}, dc[] = {0, 0, -1, 1};

    for (auto& p : positions) {
        int r = p[0], c = p[1], id = r * n + c;
        if (isLand[id]) { res.push_back(count); continue; }  // already land
        isLand[id] = true;
        ++count;                                     // new island, tentatively
        for (int d = 0; d < 4; ++d) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
            int nid = nr * n + nc;
            if (isLand[nid] && dsu.unite(id, nid)) --count;  // merged => fewer
        }
        res.push_back(count);
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(L·α(mn)) for L positions |
| Space | O(m·n) |

> Each new land cell starts as +1 island; every successful union with an already-land neighbor cancels one out — having `unite` report whether it actually merged avoids double-counting shared roots.
