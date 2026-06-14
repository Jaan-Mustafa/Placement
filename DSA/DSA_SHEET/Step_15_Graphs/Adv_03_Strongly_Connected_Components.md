# Strongly Connected Components (Kosaraju) 🔴

**Reference:** [GeeksforGeeks — Strongly Connected Components](https://www.geeksforgeeks.org/strongly-connected-components/)

---

## Problem
In a **directed** graph, a Strongly Connected Component (SCC) is a maximal set of vertices where every vertex is reachable from every other vertex in the set. Count the number of SCCs.

```
Input: n = 5, edges = [[0,1],[1,2],[2,0],[1,3],[3,4]]
Output: 3   // SCCs: {0,1,2}, {3}, {4}
```

---

## Intuition
Kosaraju's algorithm runs in three passes. **(1)** A DFS on the original graph pushes each vertex onto a stack when it finishes, ordering vertices by decreasing finish time. **(2)** Build the transpose graph (reverse every edge); this keeps SCCs intact but breaks the connections between different SCCs in one direction. **(3)** Pop vertices off the stack and run DFS on the transpose; each DFS that starts from a fresh vertex discovers exactly one whole SCC. The finish-time ordering guarantees we always begin in a "sink" SCC of the original, so the traversal cannot leak into other components.

---

## Code (C++)
```cpp
#include <vector>
#include <stack>
using namespace std;

class Kosaraju {
    void dfs1(int u, vector<vector<int>>& adj,
              vector<bool>& vis, stack<int>& st) {
        vis[u] = true;
        for (int v : adj[u])
            if (!vis[v]) dfs1(v, adj, vis, st);
        st.push(u);                      // push on finish (post-order)
    }

    void dfs2(int u, vector<vector<int>>& radj, vector<bool>& vis) {
        vis[u] = true;
        for (int v : radj[u])            // traverse transpose
            if (!vis[v]) dfs2(v, radj, vis);
    }

public:
    int countSCC(int n, vector<vector<int>>& edges) {
        vector<vector<int>> adj(n), radj(n);
        for (auto& e : edges) {
            adj[e[0]].push_back(e[1]);   // original directed edge
            radj[e[1]].push_back(e[0]);  // reversed edge (transpose)
        }

        // Pass 1: order vertices by finish time
        stack<int> st;
        vector<bool> vis(n, false);
        for (int i = 0; i < n; i++)
            if (!vis[i]) dfs1(i, adj, vis, st);

        // Pass 2 & 3: DFS transpose in finish-time order
        fill(vis.begin(), vis.end(), false);
        int sccCount = 0;
        while (!st.empty()) {
            int u = st.top(); st.pop();
            if (!vis[u]) {               // a fresh start = one new SCC
                dfs2(u, radj, vis);
                sccCount++;
            }
        }
        return sccCount;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Gotcha: the transpose DFS must follow the finish-time stack order, not numeric order — that ordering is exactly what isolates each SCC. Tarjan's algorithm finds the same SCCs in a single pass if you want to avoid building the transpose.
