# Prim's Algorithm  🟡

**Reference:** [GeeksforGeeks — Prim's Minimum Spanning Tree (MST)](https://www.geeksforgeeks.org/prims-minimum-spanning-tree-mst-greedy-algo-5/)

---

## Problem
Given a connected, undirected, weighted graph with `V` vertices, compute the total weight of its Minimum Spanning Tree using Prim's greedy algorithm.

```
Input: V=5, edges (u,v,w): (0,1,2)(0,3,6)(1,2,3)(1,3,8)(1,4,5)(2,4,7)(3,4,9)
Output: MST weight = 16   (edges 0-1, 1-2, 1-4, 0-3)
```

---

## Intuition
Prim's grows a single tree starting from any node. We keep a **min-heap** of candidate edges `{weight, node}` representing the cheapest known way to reach a not-yet-included node. We repeatedly pop the smallest. If that node is already in the tree we discard the stale entry; otherwise we pull it in, add its weight to the running sum, and push all its outgoing edges as new candidates. Because we always commit the cheapest edge crossing the current frontier (the cut property), the result is optimal. A `visited` array prevents reprocessing and cycles.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int primMST(int V, vector<vector<pair<int,int>>>& adj) {
    // min-heap of {weight, node}; greater<> makes it pop the smallest weight
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    vector<bool> visited(V, false);

    int mstWeight = 0;
    pq.push({0, 0});                      // start at node 0 with cost 0

    while (!pq.empty()) {
        auto [w, node] = pq.top(); pq.pop();
        if (visited[node]) continue;      // stale entry / would form a cycle
        visited[node] = true;
        mstWeight += w;                   // commit this edge into the MST

        for (auto& [nbr, ew] : adj[node])
            if (!visited[nbr])
                pq.push({ew, nbr});       // new frontier candidate
    }
    return mstWeight;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E log V) |
| Space | O(V + E) |

> We add the weight at pop time (not push time) so each node contributes exactly the one cheapest edge that finally pulled it in; checking `visited` after popping is what discards the stale heavier entries.
