# Dijkstra's Algorithm (Set)  🟡

**Reference:** [GeeksforGeeks — Dijkstra's Shortest Path Algorithm](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/)

---

## Problem
Same shortest-path task as the priority-queue version, but implemented with an ordered `set<pair<int,int>>` so we can **erase stale entries** instead of letting them pile up in a heap.

```
Input: V=5, edges (u,v,w): (0,1,4)(0,2,1)(2,1,2)(1,3,1)(2,3,5)(3,4,3)
Output: [0, 3, 1, 4, 7]
```

---

## Intuition
A `std::set` keeps `{dist,node}` pairs sorted, so `*set.begin()` is the closest node — just like a min-heap. The advantage is that before relaxing a neighbour to a smaller distance we can `erase` its old `{oldDist, nbr}` entry, keeping the structure free of stale duplicates. This trims memory slightly, though the asymptotic complexity is the same as the heap version. Each `insert`/`erase` is `O(log V)`.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> dijkstra(int V, vector<vector<pair<int,int>>>& adj, int src) {
    set<pair<int,int>> st;                     // {distance, node}, sorted
    vector<int> dist(V, INT_MAX);

    dist[src] = 0;
    st.insert({0, src});

    while (!st.empty()) {
        auto [d, node] = *st.begin();          // closest unfinalised node
        st.erase(st.begin());
        for (auto& [nbr, w] : adj[node]) {
            if (d + w < dist[nbr]) {
                // erase the outdated entry if one exists
                if (dist[nbr] != INT_MAX)
                    st.erase({dist[nbr], nbr});
                dist[nbr] = d + w;             // relax
                st.insert({dist[nbr], nbr});   // insert improved distance
            }
        }
    }
    return dist;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E log V) |
| Space | O(V) |

> Erasing the old pair before inserting the new one is the whole point — forget it and the set bloats with stale entries just like the heap.
