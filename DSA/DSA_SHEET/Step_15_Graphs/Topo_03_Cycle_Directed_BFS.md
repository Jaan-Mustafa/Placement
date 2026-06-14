# Detect Cycle in Directed Graph (BFS)  🟡

**Reference:** [GeeksforGeeks — Detect Cycle in a Directed Graph using BFS](https://www.geeksforgeeks.org/detect-cycle-in-a-directed-graph-using-bfs/)

---

## Problem
Given a directed graph with `V` vertices, determine whether it contains a cycle, using BFS (Kahn's algorithm).

```
Input:  V = 4, edges = [0->1, 1->2, 2->3, 3->1]
Output: true   (cycle 1 -> 2 -> 3 -> 1)
```

---

## Intuition
Kahn's topological sort can only place a node once its indegree reaches 0. In a DAG, every node eventually becomes free and gets processed, so the count of processed nodes equals `V`. If a cycle exists, the nodes on that cycle perpetually depend on each other — their indegrees never drop to 0 — so they are never enqueued. Therefore, after running Kahn's algorithm, if the number of processed nodes is less than `V`, a cycle must be present.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    bool isCyclic(int V, vector<vector<int>>& adj) {
        vector<int> indegree(V, 0);
        for (int u = 0; u < V; u++)         // build indegree array
            for (int v : adj[u])
                indegree[v]++;

        queue<int> q;
        for (int i = 0; i < V; i++)
            if (indegree[i] == 0)
                q.push(i);

        int processed = 0;                  // count of nodes placed in topo order
        while (!q.empty()) {
            int node = q.front(); q.pop();
            processed++;
            for (int next : adj[node])
                if (--indegree[next] == 0)
                    q.push(next);
        }
        return processed != V;              // not all placed => cycle exists
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> The cycle test is purely the count: `processed < V` means a cycle. No need to track recursion colours like the DFS version.
