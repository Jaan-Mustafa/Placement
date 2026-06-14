# BFS traversal  🟡

**Reference:** [GeeksforGeeks — Breadth First Search (BFS) for a Graph](https://www.geeksforgeeks.org/breadth-first-search-or-bfs-for-a-graph/)

---

## Problem
Given a graph as an adjacency list, visit every node in **Breadth-First** order starting from a source: explore all neighbours at the current distance level before moving deeper.

```
Graph:            BFS from 0:
   0 -- 1         0  ->  1, 2  ->  3, 4
   |    |
   2    3 -- 4    Output: 0 1 2 3 4
```

---

## Intuition: expand level by level using a queue
BFS uses a **FIFO queue**. Push the source and mark it visited. Repeatedly pop a node, record it, and push every **unvisited** neighbour (marking it visited *at push time*, not pop time). Because the queue holds nodes in increasing order of distance from the source, BFS naturally finds the **shortest path in an unweighted graph**.

Marking a node visited when you enqueue it (not when you dequeue) prevents the same node from being added multiple times.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

// BFS over an adjacency list starting from `src`.
vector<int> bfs(int V, vector<vector<int>>& adj, int src) {
    vector<int> order;
    vector<bool> visited(V, false);
    queue<int> q;

    visited[src] = true;        // mark on enqueue
    q.push(src);

    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);  // process the node

        for (int nbr : adj[node]) {
            if (!visited[nbr]) {
                visited[nbr] = true;   // mark before pushing -> no duplicates
                q.push(nbr);
            }
        }
    }
    return order;
}

int main() {
    int V = 5;
    vector<vector<int>> adj(V);
    auto addEdge = [&](int u, int v){ adj[u].push_back(v); adj[v].push_back(u); };
    addEdge(0,1); addEdge(0,2); addEdge(1,3); addEdge(3,4);

    for (int x : bfs(V, adj, 0)) cout << x << " ";   // 0 1 2 3 4
    cout << "\n";
    return 0;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V) for queue + visited |

> Gotcha: if the graph may be **disconnected**, wrap the BFS in a loop over all nodes (`for i in 0..V-1: if !visited[i] bfs(i)`) so unreachable components are still covered.
