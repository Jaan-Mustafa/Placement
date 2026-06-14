# Number of Operations to Make Network Connected  🟡

**Reference:** [LeetCode 1319 — Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/)

---

## Problem
There are `n` computers and a list of `connections[i] = [a, b]` cables. You may unplug a cable and replug it elsewhere. Return the minimum number of moves to connect all computers, or `-1` if impossible.

```
Input: n=4, connections=[[0,1],[0,2],[1,2]]
Output: 1   (move the redundant 1-2 cable to connect computer 3)
```

---

## Intuition
To connect `n` nodes you need at least `n-1` cables; if you have fewer it is impossible, so return `-1`. Otherwise, use a Disjoint Set to count connected **components**. Each extra cable that connects two already-connected nodes is "spare" and can be relocated. To join `c` components into one you need exactly `c-1` moves — and any valid graph with `>= n-1` edges always has enough spare cables to supply those moves. So the answer is simply `components - 1`.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    DSU(int n): parent(n), rank_(n, 0) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }
    void unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return;
        if (rank_[a] < rank_[b]) swap(a, b);
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;
    }
};

int makeConnected(int n, vector<vector<int>>& connections) {
    if ((int)connections.size() < n - 1) return -1;   // not enough cables at all
    DSU dsu(n);
    for (auto& e : connections) dsu.unite(e[0], e[1]);

    int components = 0;
    for (int i = 0; i < n; ++i)
        if (dsu.find(i) == i) ++components;           // count distinct roots
    return components - 1;                            // moves to merge them
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E·α(n) + n) |
| Space | O(n) |

> The only impossibility test is the edge-count check up front; once you have `>= n-1` cables, the spare ones are always sufficient, so the answer reduces to `components - 1`.
