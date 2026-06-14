# DFS traversal  🟡

**Reference:** [GeeksforGeeks — Depth First Search (DFS) for a Graph](https://www.geeksforgeeks.org/depth-first-search-or-dfs-for-a-graph/)

---

## Problem
Given a graph as an adjacency list, visit every node in **Depth-First** order from a source: go as deep as possible along one branch before backtracking.

```
Graph:            DFS from 0:
   0 -- 1         0 -> 1 -> 3 -> 4 (backtrack) -> 2
   |    |
   2    3 -- 4    Output: 0 1 3 4 2
```

---

## Intuition: dive deep first, recursion handles the backtracking
DFS explores one neighbour fully before the next. The simplest form is **recursion**: mark the current node visited, record it, then recurse into each unvisited neighbour. The call stack remembers where to resume, giving the natural backtracking behaviour. (Iteratively, an explicit `stack<int>` replaces the recursion.)

The exact order depends on the order neighbours appear in the adjacency list.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

// Recursive DFS helper.
void dfs(int node, vector<vector<int>>& adj, vector<bool>& visited, vector<int>& order) {
    visited[node] = true;       // mark on entry
    order.push_back(node);      // process the node

    for (int nbr : adj[node]) {
        if (!visited[nbr])
            dfs(nbr, adj, visited, order);   // recurse deeper, then backtrack
    }
}

int main() {
    int V = 5;
    vector<vector<int>> adj(V);
    auto addEdge = [&](int u, int v){ adj[u].push_back(v); adj[v].push_back(u); };
    addEdge(0,1); addEdge(0,2); addEdge(1,3); addEdge(3,4);

    vector<bool> visited(V, false);
    vector<int> order;
    dfs(0, adj, visited, order);

    for (int x : order) cout << x << " ";   // 0 1 3 4 2
    cout << "\n";
    return 0;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V) visited + O(V) recursion stack |

> Gotcha: deep or skewed graphs can blow the recursion stack (stack overflow). For very large inputs convert to an **iterative DFS** with an explicit `stack<int>`, or raise the stack limit.
