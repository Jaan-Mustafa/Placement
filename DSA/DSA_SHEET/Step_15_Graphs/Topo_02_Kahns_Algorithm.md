# Kahn's Algorithm (BFS Topo Sort)  🟡

**Reference:** [GeeksforGeeks — Topological Sorting (Indegree based / Kahn's)](https://www.geeksforgeeks.org/topological-sorting-indegree-based-solution/)

---

## Problem
Given a DAG with `V` vertices, produce a topological ordering using BFS by repeatedly removing nodes that have no incoming edges (indegree 0).

```
Input:  V = 6, edges = [5->0, 5->2, 4->0, 4->1, 2->3, 3->1]
Output: 4 5 2 0 3 1   (one valid topological order)
```

---

## Intuition
A node can be placed in the order as soon as all its prerequisites are already placed — i.e. when its indegree drops to 0. Start by enqueuing every node with indegree 0. Each time we pop a node, we "remove" it from the graph by decrementing the indegree of its neighbours; any neighbour that reaches indegree 0 becomes free and is enqueued. Processing nodes in this freeing order gives a valid topological sort. If the graph has a cycle, some nodes never reach indegree 0 and never get processed.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> topoSort(int V, vector<vector<int>>& adj) {
        vector<int> indegree(V, 0);
        for (int u = 0; u < V; u++)         // compute indegree of every node
            for (int v : adj[u])
                indegree[v]++;

        queue<int> q;
        for (int i = 0; i < V; i++)         // start with all indegree-0 nodes
            if (indegree[i] == 0)
                q.push(i);

        vector<int> order;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            order.push_back(node);
            for (int next : adj[node]) {    // "remove" node from graph
                if (--indegree[next] == 0)  // neighbour now free
                    q.push(next);
            }
        }
        return order;                       // size < V means a cycle existed
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Decrement and check indegree in one step (`--indegree[next] == 0`). If the final order has fewer than `V` nodes, the graph was not a DAG.
