# Find Eventual Safe States  🟡

**Reference:** [LeetCode 802 — Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/)

---

## Problem
Given a directed graph as `graph[i]` (list of nodes `i` points to), a node is **terminal** if it has no outgoing edges, and **safe** if every path starting from it eventually reaches a terminal node. Return all safe nodes in ascending order.

```
Input:  graph = [[1,2],[2,3],[5],[0],[5],[],[]]
Output: [2,4,5,6]
```

---

## Intuition
A node is *unsafe* exactly when it can reach a cycle. Equivalently, a node is *safe* if all of its outgoing paths lead only to terminal nodes. Reverse every edge: now terminal nodes (original outdegree 0) become nodes with indegree 0 in the reversed graph. Run Kahn's algorithm on the reversed graph — peeling off indegree-0 nodes removes terminals first, then nodes whose successors are all safe, and so on. Any node stuck in (or feeding into) a cycle never gets its reversed-indegree reduced to 0, so it is never processed and is therefore unsafe. The processed nodes are precisely the safe ones.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
        int V = graph.size();
        vector<vector<int>> rev(V);         // reversed graph
        vector<int> indegree(V, 0);         // indegree in reversed graph
        for (int u = 0; u < V; u++)
            for (int v : graph[u]) {
                rev[v].push_back(u);        // reverse edge u -> v becomes v -> u
                indegree[u]++;              // = original outdegree of u
            }

        queue<int> q;
        for (int i = 0; i < V; i++)
            if (indegree[i] == 0)           // terminal nodes (no outgoing edges)
                q.push(i);

        vector<int> safe;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            safe.push_back(node);           // node is safe
            for (int next : rev[node])      // its predecessors lose a dependency
                if (--indegree[next] == 0)
                    q.push(next);
        }

        sort(safe.begin(), safe.end());     // answer must be ascending
        return safe;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E + V log V) |
| Space | O(V + E) |

> `indegree[u]` in the reversed graph equals the *original outdegree* of `u`; terminals seed the queue. Remember to sort the result — Kahn's emits nodes in processing order, not ascending.
