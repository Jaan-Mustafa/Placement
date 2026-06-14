# Cheapest Flights Within K Stops  🟡

**Reference:** [LeetCode 787 — Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

---

## Problem
Given `n` cities and `flights[i] = [from, to, price]`, find the cheapest price from `src` to `dst` using **at most `k` stops** (i.e. at most `k+1` edges). Return `-1` if no such route exists.

```
Input: n=4, flights=[[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]],
       src=0, dst=3, k=1
Output: 700   (0->1->3, costs 100+600, within 1 stop)
```

---

## Intuition
A pure Dijkstra fails here because the cheapest route can violate the stop limit while a pricier route obeys it. Instead we do a **stops-bounded BFS**: process level by level, where each level is "one more flight". The queue holds `{stops, node, cost}`. We only relax a neighbour when we have stops remaining (`stops <= k`) and the new cost beats the best recorded cost for that node. Because we expand strictly in increasing number of stops, the stop constraint is naturally respected.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int findCheapestPrice(int n, vector<vector<int>>& flights,
                      int src, int dst, int k) {
    // adjacency list: node -> {neighbour, price}
    vector<vector<pair<int,int>>> adj(n);
    for (auto& f : flights) adj[f[0]].push_back({f[1], f[2]});

    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    queue<tuple<int,int,int>> q;          // {stops, node, cost}
    q.push({0, src, 0});

    while (!q.empty()) {
        auto [stops, node, cost] = q.front(); q.pop();
        if (stops > k) continue;          // exceeded allowed stops
        for (auto& [nbr, price] : adj[node]) {
            if (cost + price < dist[nbr] && stops <= k) {
                dist[nbr] = cost + price; // relax under stop budget
                q.push({stops + 1, nbr, dist[nbr]});
            }
        }
    }
    return dist[dst] == INT_MAX ? -1 : dist[dst];
}
```

---

## Complexity
| | |
|---|---|
| Time | O(k · E) |
| Space | O(n + E) |

> Don't use a min-heap keyed on cost here — the stop limit means a costlier-but-fewer-stops path may be the only valid one; level-order on stops is what keeps it correct.
