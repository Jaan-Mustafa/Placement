# Graph representation (adjacency matrix / list)  🟢

**Reference:** [GeeksforGeeks — Graph and its representations](https://www.geeksforgeeks.org/graph-and-its-representations/)

---

## Problem
Store a graph in memory so algorithms can query it efficiently. The two standard ways are the **adjacency matrix** and the **adjacency list**. Given an edge list, build both.

```
V = 4, edges = (0,1) (0,2) (1,2) (2,3)   (undirected)

Adjacency matrix          Adjacency list
   0 1 2 3                 0 -> 1, 2
0  0 1 1 0                 1 -> 0, 2
1  1 0 1 0                 2 -> 0, 1, 3
2  1 1 0 1                 3 -> 2
3  0 0 1 0
```

---

## Intuition: matrix is O(1) lookup but O(V²) memory; list is sparse-friendly
- **Adjacency matrix** — a `V x V` grid where `mat[u][v] = 1` (or the weight) if an edge exists. Edge existence check is **O(1)**, but it always costs **O(V²)** space, wasteful for sparse graphs.
- **Adjacency list** — for each node, a list of its neighbours. Space is **O(V + E)**, which is ideal for the sparse graphs common in interviews. Checking a specific edge takes O(degree) instead of O(1).

Use the **list** (`vector<vector<int>>`) by default; reach for the matrix only when the graph is dense or you need constant-time edge queries.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int V = 4;
    vector<pair<int,int>> edges = {{0,1},{0,2},{1,2},{2,3}};

    // ---- Adjacency matrix (V x V), undirected ----
    vector<vector<int>> mat(V, vector<int>(V, 0));
    for (auto &e : edges) {
        mat[e.first][e.second] = 1;
        mat[e.second][e.first] = 1;         // mirror for undirected
    }

    // ---- Adjacency list (preferred) ----
    vector<vector<int>> adj(V);
    for (auto &e : edges) {
        adj[e.first].push_back(e.second);
        adj[e.second].push_back(e.first);   // mirror for undirected
    }

    // O(1) edge query via matrix
    cout << "edge(0,2)? " << (mat[0][2] ? "yes" : "no") << "\n";

    // iterate neighbours via list
    for (int u = 0; u < V; u++) {
        cout << u << " -> ";
        for (int v : adj[u]) cout << v << " ";
        cout << "\n";
    }
    return 0;
}
```

---

## Complexity
| | |
|---|---|
| Time | Build matrix O(V²) init + O(E); build list O(E) |
| Space | Matrix O(V²) · List O(V + E) |

> Gotcha: for a **weighted** graph store `mat[u][v] = w` (and an adjacency list of `pair<int,int>` = `{neighbour, weight}`); use a sentinel like `INF`/`0` consistently so "no edge" is never confused with a zero-weight edge.
