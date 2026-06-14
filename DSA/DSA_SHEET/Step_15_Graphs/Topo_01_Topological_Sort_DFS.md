# Topological Sort (DFS)  🟡

**Reference:** [GeeksforGeeks — Topological Sorting](https://www.geeksforgeeks.org/topological-sorting/)

---

## Problem
Given a Directed Acyclic Graph (DAG) with `V` vertices, return a linear ordering of vertices such that for every directed edge `u -> v`, `u` appears before `v` in the ordering.

```
Input:  V = 6, edges = [5->0, 5->2, 4->0, 4->1, 2->3, 3->1]
Output: 5 4 2 3 1 0   (one valid topological order)
```

---

## Intuition
In a topological order, a node must come before all the nodes it points to. With DFS, once we finish exploring every descendant of a node, that node has no remaining dependencies left to place — so it is "done" and can be pushed onto a stack. Because we push a node only after all its children are fully processed, popping the stack (i.e. reversing the finish order) yields an order where every node precedes its descendants. This works only on a DAG; a cycle would have no valid ordering.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    void dfs(int node, vector<vector<int>>& adj,
             vector<int>& visited, stack<int>& st) {
        visited[node] = 1;
        for (int next : adj[node])          // explore all neighbours first
            if (!visited[next])
                dfs(next, adj, visited, st);
        st.push(node);                      // push only after children done
    }

public:
    vector<int> topoSort(int V, vector<vector<int>>& adj) {
        vector<int> visited(V, 0);
        stack<int> st;
        for (int i = 0; i < V; i++)         // graph may be disconnected
            if (!visited[i])
                dfs(i, adj, visited, st);

        vector<int> order;                  // pop stack = reverse finish time
        while (!st.empty()) {
            order.push_back(st.top());
            st.pop();
        }
        return order;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> The stack reverses the finish order for free — never sort by finish time manually. Valid only on a DAG; on a cyclic graph the output is meaningless.
