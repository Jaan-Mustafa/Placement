# Bipartite Graph (BFS / DFS)  🟡

**Reference:** [LeetCode 785 — Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)

---

## Problem
Given an undirected graph as an adjacency list, decide whether it is bipartite — i.e. its vertices can be split into two sets such that every edge connects a vertex from one set to the other.

```
Input: graph = [[1,3],[0,2],[1,3],[0,2]]
Output: true     // {0,2} and {1,3}
```

---

## Intuition
A graph is bipartite iff it is 2-colorable. Traverse the graph coloring each node, alternating colors between a node and its neighbours. Using BFS: color the start node `0`, then every neighbour gets the opposite color. If we ever find a neighbour already colored the **same** as the current node, the two-coloring fails and the graph is not bipartite (equivalently, it contains an odd-length cycle). Repeat per component for disconnected graphs.

---

## Code (C++)
```cpp
class Solution {
public:
    bool bfsCheck(int start, vector<vector<int>>& graph, vector<int>& color) {
        queue<int> q;
        q.push(start);
        color[start] = 0;                       // first color

        while (!q.empty()) {
            int node = q.front(); q.pop();
            for (int nbr : graph[node]) {
                if (color[nbr] == -1) {         // uncolored -> opposite color
                    color[nbr] = 1 - color[node];
                    q.push(nbr);
                } else if (color[nbr] == color[node]) {
                    return false;               // same-color edge -> not bipartite
                }
            }
        }
        return true;
    }

    bool isBipartite(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> color(n, -1);               // -1 = uncolored
        for (int i = 0; i < n; i++)
            if (color[i] == -1 && !bfsCheck(i, graph, color))
                return false;                   // some component failed
        return true;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V) |

> Bipartite is exactly equivalent to "no odd-length cycle" — a same-color edge is precisely the moment an odd cycle is detected.
