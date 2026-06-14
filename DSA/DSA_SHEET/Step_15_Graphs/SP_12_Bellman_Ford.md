# Bellman-Ford Algorithm  🔴

**Reference:** [GeeksforGeeks — Bellman-Ford Algorithm](https://www.geeksforgeeks.org/bellman-ford-algorithm-dp-23/)

---

## Problem
Given a directed weighted graph (edges may be **negative**) with `V` nodes and a source `src`, find shortest distances from `src` to all nodes. Also **detect a negative-weight cycle**; if one exists, report it.

```
Input: V=5, edges (u,v,w): (0,1,-1)(0,2,4)(1,2,3)(1,3,2)(1,4,2)(3,2,5)(3,1,1)(4,3,-3)
Output: [0, -1, 2, -2, 1]
```

---

## Intuition
Unlike Dijkstra, Bellman-Ford handles negative edges. A shortest path uses at most `V-1` edges, so relaxing **all** edges `V-1` times guarantees every shortest distance settles. If a `V`-th relaxation pass can still improve some distance, a **negative cycle** is reachable (its loop keeps lowering the cost), so no finite shortest path exists.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

// edges given as {u, v, w}
vector<int> bellmanFord(int V, vector<vector<int>>& edges, int src) {
    const int INF = 1e8;
    vector<int> dist(V, INF);
    dist[src] = 0;

    // relax all edges V-1 times
    for (int i = 0; i < V - 1; ++i) {
        for (auto& e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != INF && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;        // relax edge u->v
        }
    }

    // V-th pass: any further relaxation => negative cycle
    for (auto& e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] != INF && dist[u] + w < dist[v])
            return {-1};                       // negative cycle detected
    }
    return dist;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(V · E) |
| Space | O(V) |

> Guard every relaxation with `dist[u] != INF`; without it `INF + w` overflows and silently corrupts unreachable nodes into looking reachable.
