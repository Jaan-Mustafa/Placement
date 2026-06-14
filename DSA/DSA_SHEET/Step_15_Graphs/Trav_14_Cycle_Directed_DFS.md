# Detect Cycle in Directed Graph (DFS)  🟡

**Reference:** [GeeksforGeeks — Detect Cycle in a Directed Graph](https://www.geeksforgeeks.org/detect-cycle-in-a-graph/)

---

## Problem
Given a directed graph with `V` vertices and an adjacency list, determine whether it contains a cycle.

```
Input: V = 4, edges = [[0,1],[1,2],[2,3],[3,1]]
Output: true     // 1 -> 2 -> 3 -> 1
```

---

## Intuition
The undirected parent trick does not work for directed graphs because direction matters. Instead we track two states per node: `visited` (ever explored) and `pathVisited` (currently on the active DFS recursion stack). A cycle exists iff DFS reaches a node that is already on the current path — that's a back-edge into an ancestor. When a DFS call returns, we clear its `pathVisited` flag so it no longer counts as part of other paths.

---

## Code (C++)
```cpp
class Solution {
public:
    bool dfs(int node, vector<vector<int>>& adj, vector<int>& vis, vector<int>& pathVis) {
        vis[node] = 1;
        pathVis[node] = 1;                      // node is on the current path

        for (int nbr : adj[node]) {
            if (!vis[nbr]) {
                if (dfs(nbr, adj, vis, pathVis)) return true;
            } else if (pathVis[nbr]) {
                return true;                    // back-edge to an active ancestor
            }
        }

        pathVis[node] = 0;                      // backtrack: leave the path
        return false;
    }

    bool isCyclic(int V, vector<vector<int>>& adj) {
        vector<int> vis(V, 0), pathVis(V, 0);
        for (int i = 0; i < V; i++)
            if (!vis[i] && dfs(i, adj, vis, pathVis))
                return true;
        return false;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V) |

> A node being `visited` but **not** on the current path is fine (cross/forward edge); only a neighbour with `pathVisited == 1` signals a true directed cycle.
