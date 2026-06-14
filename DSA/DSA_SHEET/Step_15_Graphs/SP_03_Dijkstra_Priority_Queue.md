# Dijkstra's Algorithm (Priority Queue)  🟡

**Reference:** [GeeksforGeeks — Dijkstra's Shortest Path Algorithm](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/)

---

## Problem
Given a weighted graph with **non-negative** edge weights, `V` nodes, and a source `src`, compute the shortest distance from `src` to every node.

```
Input: V=5, edges (u,v,w): (0,1,4)(0,2,1)(2,1,2)(1,3,1)(2,3,5)(3,4,3)
Output: [0, 3, 1, 4, 7]
```

---

## Intuition
Dijkstra greedily finalises the closest unvisited node. A min-heap (priority queue) keyed on distance lets us always pop the currently nearest node in `O(log V)`. We push `{newDist, neighbour}` whenever we relax an edge; stale larger entries are simply ignored when popped because their distance no longer matches `dist[]`. Because all weights are non-negative, once a node is popped its distance is final.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> dijkstra(int V, vector<vector<pair<int,int>>>& adj, int src) {
    // min-heap of {distance, node}
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    vector<int> dist(V, INT_MAX);

    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (d > dist[node]) continue;          // skip stale entry
        for (auto& [nbr, w] : adj[node]) {
            if (d + w < dist[nbr]) {           // relax edge
                dist[nbr] = d + w;
                pq.push({dist[nbr], nbr});     // push improved distance
            }
        }
    }
    return dist;                               // INT_MAX => unreachable
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E log V) |
| Space | O(V + E) |

> Dijkstra breaks on negative edges — a node finalised early might later be reachable more cheaply; use Bellman-Ford instead when negatives exist.
