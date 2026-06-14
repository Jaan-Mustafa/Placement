# Number of Provinces  🟡

**Reference:** [LeetCode 547 — Number of Provinces](https://leetcode.com/problems/number-of-provinces/)

---

## Problem
Given an `n x n` adjacency matrix `isConnected` where `isConnected[i][j] = 1` means city `i` and city `j` are directly connected, return the number of provinces (groups of directly or indirectly connected cities).

```
Input: isConnected = [[1,1,0],[1,1,0],[0,0,1]]
Output: 2     // {0,1} form one province, {2} another
```

---

## Intuition
A province is just a connected component of an undirected graph. Build (or use directly) the adjacency information, then run a traversal (DFS/BFS) from every unvisited node. Each time we start a fresh traversal we have discovered a brand-new component, so we increment a counter. The matrix is treated as a dense adjacency representation: row `i` lists which nodes `i` connects to.

---

## Code (C++)
```cpp
class Solution {
public:
    void dfs(int node, vector<vector<int>>& adj, vector<int>& vis) {
        vis[node] = 1;                          // mark current city visited
        for (int next = 0; next < adj.size(); next++) {
            // edge exists and the neighbour is not yet visited
            if (adj[node][next] == 1 && !vis[next])
                dfs(next, adj, vis);
        }
    }

    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        vector<int> vis(n, 0);                  // visited flags
        int provinces = 0;
        for (int i = 0; i < n; i++) {
            if (!vis[i]) {                      // new unvisited city -> new province
                provinces++;
                dfs(i, isConnected, vis);       // sink the whole component
            }
        }
        return provinces;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N^2) |
| Space | O(N) |

> The matrix is already the adjacency structure — no need to build an extra list. Counting provinces == counting connected components.
