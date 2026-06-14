# Detect Cycle in Undirected Graph (BFS)  🟡

**Reference:** [GeeksforGeeks — Detect Cycle in an Undirected Graph](https://www.geeksforgeeks.org/detect-cycle-in-an-undirected-graph/)

---

## Problem
Given an undirected graph with `V` vertices and an adjacency list, determine whether it contains any cycle.

```
Input: V = 4, edges = [[0,1],[1,2],[2,3],[3,0]]
Output: true     // 0-1-2-3-0 is a cycle
```

---

## Intuition
In an undirected graph, every edge can be traversed both ways, so simply reaching a visited node is not enough — we might just be looking back at the parent we came from. The trick: during BFS, carry each node's parent. If we encounter an already-visited neighbour that is **not** the parent, we have reached the same node by two different paths, which means a cycle exists. We must launch BFS from every unvisited node to cover disconnected components.

---

## Code (C++)
```cpp
class Solution {
public:
    bool bfsCheck(int start, vector<vector<int>>& adj, vector<int>& vis) {
        queue<pair<int,int>> q;                 // {node, parent}
        q.push({start, -1});
        vis[start] = 1;

        while (!q.empty()) {
            auto [node, parent] = q.front(); q.pop();
            for (int nbr : adj[node]) {
                if (!vis[nbr]) {
                    vis[nbr] = 1;
                    q.push({nbr, node});        // remember where we came from
                } else if (nbr != parent) {
                    return true;                // visited & not parent -> cycle
                }
            }
        }
        return false;
    }

    bool isCycle(int V, vector<vector<int>>& adj) {
        vector<int> vis(V, 0);
        for (int i = 0; i < V; i++)
            if (!vis[i] && bfsCheck(i, adj, vis))
                return true;                    // cycle in some component
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

> Tracking the parent is what distinguishes a real cycle from the harmless back-edge to the node you just came from.
