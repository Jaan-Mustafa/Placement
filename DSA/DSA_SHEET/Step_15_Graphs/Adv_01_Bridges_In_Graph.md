# Bridges in Graph (Tarjan) 🔴

**Reference:** [LeetCode 1192 — Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/)

---

## Problem
Given `n` servers (nodes `0..n-1`) connected by undirected `connections`, a **critical connection** (bridge) is an edge whose removal disconnects some servers from the rest. Return all such edges.

```
Input: n = 4, connections = [[0,1],[1,2],[2,0],[1,3]]
Output: [[1,3]]   // removing 1-3 isolates server 3
```

---

## Intuition
Run a DFS assigning each node a discovery time `disc[u]` (the order it was first visited). `low[u]` is the earliest discovery time reachable from `u` using its subtree plus at most one back-edge. While exploring a tree edge `(u, v)`, recurse into `v` and then relax `low[u] = min(low[u], low[v])`. If after recursion `low[v] > disc[u]`, then `v`'s subtree has no back-edge climbing to `u` or above — so the only path connecting them is this edge, making it a bridge. We must ignore the immediate parent edge so a single edge is not mistaken for a back-edge.

---

## Code (C++)
```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
    int timer = 0;                       // global discovery counter
    vector<int> disc, low;               // disc[u]=discovery time, low[u]=earliest reachable
    vector<vector<int>> adj;             // adjacency list
    vector<vector<int>> bridges;         // result edges

    void dfs(int u, int parent) {
        disc[u] = low[u] = timer++;      // assign discovery time, init low
        for (int v : adj[u]) {
            if (v == parent) continue;   // skip the edge we came from
            if (disc[v] == -1) {         // v unvisited -> tree edge
                dfs(v, u);
                low[u] = min(low[u], low[v]);     // propagate child's low up
                if (low[v] > disc[u])             // no back-edge reaches u or higher
                    bridges.push_back({u, v});    // (u,v) is a bridge
            } else {                     // v already visited -> back edge
                low[u] = min(low[u], disc[v]);    // can climb to disc[v]
            }
        }
    }

public:
    vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
        adj.assign(n, {});
        disc.assign(n, -1);              // -1 marks unvisited
        low.assign(n, 0);
        for (auto& e : connections) {    // build undirected graph
            adj[e[0]].push_back(e[1]);
            adj[e[1]].push_back(e[0]);
        }
        for (int i = 0; i < n; i++)      // handle possibly disconnected graph
            if (disc[i] == -1) dfs(i, -1);
        return bridges;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Gotcha: use `low[v] > disc[u]` for bridges (strict `>`). With parallel edges, skipping only the single parent edge is insufficient — track the edge id instead of the parent node.
