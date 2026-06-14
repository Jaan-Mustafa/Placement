# Shortest Path in Undirected Graph (Unit Weights)  🟡

**Reference:** [GeeksforGeeks — Shortest Path in Unweighted Graph](https://www.geeksforgeeks.org/shortest-path-unweighted-graph/)

---

## Problem
Given an undirected graph with `V` nodes and unit-weight edges, find the shortest distance from a source node `src` to every other node. If a node is unreachable, mark its distance as `-1`.

```
Input: V=6, edges=[[0,1],[0,3],[1,2],[3,4],[4,5]], src=0
Output: [0, 1, 2, 1, 2, 3]
```

---

## Intuition
When every edge has the same weight, the number of edges on a path equals its cost. Plain BFS explores the graph in expanding "rings" of increasing distance from the source, so the first time we reach a node is guaranteed to be along a shortest path. We keep a `dist[]` array initialised to infinity, set `dist[src]=0`, and relax neighbours level by level — no priority queue is needed because all edges are equal.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> shortestPath(int V, vector<vector<pair<int,int>>>& adj, int src) {
    vector<int> dist(V, INT_MAX);     // distance from src, INF initially
    dist[src] = 0;
    queue<int> q;
    q.push(src);

    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (auto& [nbr, w] : adj[node]) {   // w == 1 for unit weights
            // first BFS arrival is the shortest in an unweighted graph
            if (dist[node] + 1 < dist[nbr]) {
                dist[nbr] = dist[node] + 1;
                q.push(nbr);
            }
        }
    }

    for (int& d : dist) if (d == INT_MAX) d = -1;  // unreachable -> -1
    return dist;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V) |

> BFS only gives shortest paths when all edge weights are equal; the moment weights differ you must switch to Dijkstra.
