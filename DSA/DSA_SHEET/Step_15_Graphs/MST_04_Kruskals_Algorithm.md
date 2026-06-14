# Kruskal's Algorithm  🟡

**Reference:** [GeeksforGeeks — Kruskal's Minimum Spanning Tree Algorithm](https://www.geeksforgeeks.org/kruskals-minimum-spanning-tree-algorithm-greedy-algo-2/)

---

## Problem
Given a connected, undirected, weighted graph as an edge list, compute the total weight of its Minimum Spanning Tree using Kruskal's greedy algorithm.

```
Input: V=4, edges (w,u,v): (1,0,1)(2,1,2)(3,2,3)(4,0,3)(5,0,2)
Output: MST weight = 6   (edges 0-1, 1-2, 2-3; the 4 and 5 edges form cycles)
```

---

## Intuition
Sort every edge by weight ascending, then walk through them cheapest-first. For each edge, if its two endpoints are already in the same set, adding it would create a cycle, so skip it; otherwise it safely connects two previously separate components, so we `unite` them and add the weight. A Disjoint Set answers "same component?" in near-constant time. After processing we will have added exactly `V-1` edges — the MST.

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

int kruskalMST(int V, vector<array<int,3>>& edges) {  // edges: {weight, u, v}
    sort(edges.begin(), edges.end());                 // sort by weight ascending
    DSU dsu(V);
    int mstWeight = 0, used = 0;

    for (auto& [w, u, v] : edges) {
        if (dsu.find(u) != dsu.find(v)) {             // adding it makes no cycle
            dsu.unite(u, v);
            mstWeight += w;
            if (++used == V - 1) break;               // MST complete
        }
    }
    return mstWeight;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E log E) |
| Space | O(V + E) |

> The dominant cost is sorting the edges; the DSU work is effectively linear, so Kruskal's shines on sparse graphs where E is small relative to V².
