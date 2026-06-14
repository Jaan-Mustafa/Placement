# Minimum Spanning Tree (Theory)  🟡

**Reference:** [GeeksforGeeks — What is Minimum Spanning Tree (MST)](https://www.geeksforgeeks.org/what-is-minimum-spanning-tree-mst/)

---

## Problem
Given a connected, undirected, weighted graph with `V` vertices and `E` edges, find a **spanning tree** (a subgraph that connects all vertices) whose total edge weight is minimum.

```
Input: V=4, edges (u,v,w): (0,1,1)(1,2,2)(2,3,3)(0,3,4)(0,2,5)
A spanning tree picks V-1 = 3 edges connecting all nodes.
Output: MST weight = 1 + 2 + 3 = 6   (edges 0-1, 1-2, 2-3)
```

---

## Intuition
A **spanning tree** of a graph with `V` nodes is a connected, acyclic subgraph that touches every vertex using exactly `V-1` edges. A graph can have many spanning trees; the **minimum** spanning tree is the one whose summed edge weight is the smallest. Key properties: it always has exactly `V-1` edges, it contains no cycle (adding any non-tree edge creates exactly one cycle), and it spans all vertices. Two classic greedy algorithms build it: **Prim's** grows a single tree outward from a start node by repeatedly absorbing the cheapest edge crossing the frontier, while **Kruskal's** sorts all edges and adds the cheapest that does not form a cycle (tracked with a Disjoint Set). Both rely on the **cut property**: the lightest edge crossing any partition of the vertices is safe to include in some MST.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

// Pseudo-skeletons of the two greedy strategies (full versions on later pages).

// PRIM: grow one tree using a min-heap keyed by edge weight.
//   push {0, start}; while heap: pop cheapest {w, node};
//   if node already in tree skip; else add w to total, mark node,
//   push every outgoing edge {weight, neighbour}.

// KRUSKAL: pick globally cheapest safe edges.
//   sort edges by weight ascending;
//   for each edge (u,v,w): if find(u) != find(v) (no cycle)
//        unite(u,v) and add w to total.

int main() {
    // Example: total MST weight is the same regardless of which algorithm runs.
    cout << "MST has exactly V-1 edges, no cycle, minimum total weight\n";
    return 0;
}
```

---

## Complexity
| | |
|---|---|
| Prim's (min-heap) | Time O(E log V), Space O(V + E) |
| Kruskal's (sort + DSU) | Time O(E log E), Space O(V + E) |

> Both are greedy and both are correct because of the cut property; pick Prim's for dense graphs (adjacency-driven) and Kruskal's for sparse, edge-list graphs.
