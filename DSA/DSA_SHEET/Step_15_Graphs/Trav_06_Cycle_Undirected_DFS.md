# Detect Cycle in Undirected Graph (DFS)  🟡

**Reference:** [GeeksforGeeks — Detect Cycle in an Undirected Graph (DFS)](https://www.geeksforgeeks.org/detect-cycle-undirected-graph/)

---

## Problem
Given an undirected graph with `V` vertices and an adjacency list, determine whether it contains any cycle — using DFS this time.

```
Input: V = 3, edges = [[0,1],[1,2],[2,0]]
Output: true     // 0-1-2-0 is a cycle
```

---

## Intuition
Same idea as the BFS version but with recursion. Each DFS call receives the node and the parent it was reached from. We mark the node visited and explore its neighbours. If a neighbour is unvisited, recurse with the current node as parent. If a neighbour is already visited and is **not** the parent, we found a second path into a visited node — that closes a cycle. Run DFS from every unvisited node so disconnected components are all checked.

---

## Code (C++)
```cpp
class Solution {
public:
    bool dfs(int node, int parent, vector<vector<int>>& adj, vector<int>& vis) {
        vis[node] = 1;
        for (int nbr : adj[node]) {
            if (!vis[nbr]) {
                if (dfs(nbr, node, adj, vis)) return true;  // cycle found deeper
            } else if (nbr != parent) {
                return true;                                 // back-edge -> cycle
            }
        }
        return false;
    }

    bool isCycle(int V, vector<vector<int>>& adj) {
        vector<int> vis(V, 0);
        for (int i = 0; i < V; i++)
            if (!vis[i] && dfs(i, -1, adj, vis))
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

> The recursion stack depth can reach O(V) on a path-shaped graph — that, plus the visited array, is the space cost.
