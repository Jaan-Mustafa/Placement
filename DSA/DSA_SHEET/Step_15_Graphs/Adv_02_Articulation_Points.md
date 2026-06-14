# Articulation Points 🔴

**Reference:** [GeeksforGeeks — Articulation Points (or Cut Vertices) in a Graph](https://www.geeksforgeeks.org/articulation-points-or-cut-vertices-in-a-graph/)

---

## Problem
In an undirected graph, an **articulation point** (cut vertex) is a node whose removal (with its edges) increases the number of connected components. Return the set of all such vertices.

```
Input: n = 5, edges = [[0,1],[1,2],[2,0],[1,3],[3,4]]
Output: {1, 3}   // removing 1 isolates {3,4}; removing 3 isolates 4
```

---

## Intuition
Like the bridge algorithm, DFS computes `disc[u]` and `low[u]`. For a **non-root** node `u` with a DFS child `v`, if `low[v] >= disc[u]`, then `v`'s subtree cannot reach any ancestor of `u` without passing through `u` — so removing `u` disconnects that subtree, making `u` an articulation point. The condition uses `>=` (not strict) because even staying at `u` itself counts. The **root** is special: it is an articulation point only if it has more than one DFS child, since otherwise its single subtree stays connected after removal.

---

## Code (C++)
```cpp
#include <vector>
#include <set>
#include <algorithm>
using namespace std;

class ArticulationPoints {
    int timer = 0;
    vector<int> disc, low;
    vector<bool> isAP;                   // marks articulation points
    vector<vector<int>> adj;

    void dfs(int u, int parent) {
        disc[u] = low[u] = timer++;
        int children = 0;                // count of DFS-tree children of u
        for (int v : adj[u]) {
            if (v == parent) continue;   // skip parent edge
            if (disc[v] == -1) {         // tree edge
                children++;
                dfs(v, u);
                low[u] = min(low[u], low[v]);
                // non-root cut vertex: child cannot escape above u
                if (parent != -1 && low[v] >= disc[u])
                    isAP[u] = true;
            } else {                     // back edge
                low[u] = min(low[u], disc[v]);
            }
        }
        // root is a cut vertex iff it has > 1 DFS children
        if (parent == -1 && children > 1)
            isAP[u] = true;
    }

public:
    set<int> findArticulationPoints(int n, vector<vector<int>>& edges) {
        adj.assign(n, {});
        disc.assign(n, -1);
        low.assign(n, 0);
        isAP.assign(n, false);
        for (auto& e : edges) {          // undirected graph
            adj[e[0]].push_back(e[1]);
            adj[e[1]].push_back(e[0]);
        }
        for (int i = 0; i < n; i++)      // cover all components
            if (disc[i] == -1) dfs(i, -1);
        set<int> result;
        for (int i = 0; i < n; i++)
            if (isAP[i]) result.insert(i);
        return result;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Gotcha: articulation uses `low[v] >= disc[u]` (non-strict, the opposite of the bridge condition), and the root needs the separate `children > 1` rule. A single node may be flagged from multiple children — use a boolean array so it is reported once.
