# Graph and its types  🟢

**Reference:** [GeeksforGeeks — Graph and its representations](https://www.geeksforgeeks.org/graph-and-its-representations/)

---

## Problem
A **graph** is a non-linear data structure made of a set of **vertices (nodes)** connected by **edges**. Understand the vocabulary and the common classifications before touching any algorithm.

```
Nodes (V): {0, 1, 2, 3}
Edges (E): {(0,1), (0,2), (1,2), (2,3)}

      0 ---- 1
      |    /
      |  /
      2 ---- 3
```

---

## Intuition: classify by direction, weight, cycles and connectivity
A graph is described along a few independent axes:

- **Undirected vs Directed:** an undirected edge `(u, v)` works both ways; a directed edge `u → v` is one-way (used in DAGs, dependency graphs).
- **Weighted vs Unweighted:** edges may carry a cost/weight (Dijkstra, MST) or just represent connectivity.
- **Cyclic vs Acyclic:** a cycle is a path that starts and ends at the same node. A directed acyclic graph (**DAG**) has no directed cycle and supports topological sorting.
- **Connected vs Disconnected:** in a connected graph every node is reachable from every other; otherwise it splits into multiple **components**.
- **Degree:** number of edges touching a node. For undirected graphs, the sum of all degrees equals `2 * E`. For directed graphs each node has an **indegree** and **outdegree**.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

// Build an UNDIRECTED graph as an adjacency list and report basic facts.
int main() {
    int V = 4, E = 4;                       // 4 vertices, 4 edges
    vector<vector<int>> adj(V);             // adj[u] = neighbours of u

    // edge list for the picture above
    vector<pair<int,int>> edges = {{0,1},{0,2},{1,2},{2,3}};

    for (auto &e : edges) {
        int u = e.first, v = e.second;
        adj[u].push_back(v);                // u -> v
        adj[v].push_back(u);                // v -> u  (undirected: add both)
    }

    // degree of each node = size of its adjacency list
    for (int u = 0; u < V; u++)
        cout << "degree(" << u << ") = " << adj[u].size() << "\n";

    // sum of degrees == 2 * E for an undirected graph (handshake lemma)
    int sumDeg = 0;
    for (int u = 0; u < V; u++) sumDeg += adj[u].size();
    cout << "sum of degrees = " << sumDeg << " == 2*E = " << 2 * E << "\n";
    return 0;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) to build / scan |
| Space | O(V + E) for the adjacency list |

> Gotcha: for a **directed** edge `u → v` push only `adj[u].push_back(v)` — adding the reverse silently turns your graph undirected and breaks topological / cycle-detection logic.
