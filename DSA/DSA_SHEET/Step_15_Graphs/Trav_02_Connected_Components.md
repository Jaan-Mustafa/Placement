# Connected Components  🟡

**Reference:** [GeeksforGeeks — Connected Components in an Undirected Graph](https://www.geeksforgeeks.org/connected-components-in-an-undirected-graph/)

---

## Problem
Given an undirected graph with `V` vertices (`0..V-1`) and a list of edges, count the number of connected components — maximal sets of vertices reachable from one another.

```
Input: V = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2     // {0,1,2} and {3,4}
```

---

## Intuition
Build an adjacency list from the edge list. Then iterate over all vertices; whenever we meet a vertex that has not been visited, it belongs to a new component, so we increment the count and flood the entire reachable region with DFS (or BFS) to mark it visited. Isolated vertices with no edges still count as their own component because the outer loop visits every vertex.

---

## Code (C++)
```cpp
class Solution {
public:
    void dfs(int node, vector<vector<int>>& adj, vector<int>& vis) {
        vis[node] = 1;
        for (int nbr : adj[node])
            if (!vis[nbr]) dfs(nbr, adj, vis);
    }

    int countComponents(int V, vector<vector<int>>& edges) {
        // build adjacency list (undirected -> add both directions)
        vector<vector<int>> adj(V);
        for (auto& e : edges) {
            adj[e[0]].push_back(e[1]);
            adj[e[1]].push_back(e[0]);
        }

        vector<int> vis(V, 0);
        int components = 0;
        for (int i = 0; i < V; i++) {
            if (!vis[i]) {                  // unseen vertex -> new component
                components++;
                dfs(i, adj, vis);
            }
        }
        return components;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Every vertex is visited exactly once across all DFS calls — the outer loop only triggers a traversal for component roots.
