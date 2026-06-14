# Shortest Path in DAG (Topo Sort)  🟡

**Reference:** [GeeksforGeeks — Shortest Path for Directed Acyclic Graphs](https://www.geeksforgeeks.org/shortest-path-for-directed-acyclic-graphs/)

---

## Problem
Given a weighted Directed Acyclic Graph (DAG) with `V` nodes, find the shortest distance from source `0` to all other nodes. Unreachable nodes are reported as `-1`.

```
Input: V=6, edges (u,v,w): (0,1,2)(0,4,1)(4,5,4)(4,2,2)(1,2,3)(2,3,6)(5,3,1)
Output: [0, 2, 3, 6, 1, 5]
```

---

## Intuition
In a DAG there are no cycles, so we can order the vertices linearly via topological sort such that every edge `u -> v` points "forward". If we relax edges strictly in topological order, then by the time we process a node its own shortest distance is already final (all predecessors were handled earlier). This avoids the need for a priority queue and even works with negative edge weights.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

void topoDFS(int node, vector<vector<pair<int,int>>>& adj,
             vector<bool>& vis, stack<int>& st) {
    vis[node] = true;
    for (auto& [nbr, w] : adj[node])
        if (!vis[nbr]) topoDFS(nbr, adj, vis, st);
    st.push(node);                       // post-order push => topo order on pop
}

vector<int> shortestPathDAG(int V, vector<vector<pair<int,int>>>& adj, int src) {
    vector<bool> vis(V, false);
    stack<int> st;
    for (int i = 0; i < V; ++i)
        if (!vis[i]) topoDFS(i, adj, vis, st);   // build topological order

    vector<int> dist(V, INT_MAX);
    dist[src] = 0;

    while (!st.empty()) {                 // process nodes in topo order
        int node = st.top(); st.pop();
        if (dist[node] == INT_MAX) continue;       // unreachable so far, skip
        for (auto& [nbr, w] : adj[node])
            if (dist[node] + w < dist[nbr])         // relax forward edge
                dist[nbr] = dist[node] + w;
    }

    for (int& d : dist) if (d == INT_MAX) d = -1;
    return dist;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V) |

> Topo-order relaxation handles negative weights for free — but only because a DAG has no cycles; on a general graph you'd need Bellman-Ford.
