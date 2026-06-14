# Number of Ways to Arrive at Destination  🟡

**Reference:** [LeetCode 1976 — Number of Ways to Arrive at Destination](https://leetcode.com/problems/number-of-ways-to-arrive-at-destination/)

---

## Problem
In a city of `n` intersections (`0` to `n-1`) connected by bidirectional roads `roads[i] = [u, v, time]`, count the number of distinct **shortest-time** paths from intersection `0` to `n-1`. Return the count modulo `1e9+7`.

```
Input: n=7, roads=[[0,6,7],[0,1,2],[1,2,3],[1,3,3],[6,3,3],
       [3,5,1],[6,5,1],[2,5,1],[0,4,5],[4,6,2]]
Output: 4
```

---

## Intuition
Run Dijkstra while also tracking `ways[v]`, the number of shortest paths to `v`. When we find a **strictly shorter** distance to a neighbour, the way-count is copied from the predecessor (`ways[nbr] = ways[node]`). When we find an **equal** distance via a different edge, we **add** the predecessor's count (`ways[nbr] += ways[node]`), all under modulo arithmetic.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int countPaths(int n, vector<vector<int>>& roads) {
    const int MOD = 1e9 + 7;
    vector<vector<pair<int,int>>> adj(n);   // {neighbour, time}
    for (auto& r : roads) {
        adj[r[0]].push_back({r[1], r[2]});
        adj[r[1]].push_back({r[0], r[2]});  // bidirectional
    }

    vector<long long> dist(n, LLONG_MAX);
    vector<long long> ways(n, 0);
    priority_queue<pair<long long,int>,
                   vector<pair<long long,int>>, greater<>> pq;
    dist[0] = 0;
    ways[0] = 1;
    pq.push({0, 0});

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (d > dist[node]) continue;       // stale entry
        for (auto& [nbr, w] : adj[node]) {
            if (d + w < dist[nbr]) {        // strictly shorter -> inherit count
                dist[nbr] = d + w;
                ways[nbr] = ways[node];
                pq.push({dist[nbr], nbr});
            } else if (d + w == dist[nbr]) { // equal -> accumulate count
                ways[nbr] = (ways[nbr] + ways[node]) % MOD;
            }
        }
    }
    return (int) ways[n - 1];
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E log V) |
| Space | O(V + E) |

> Use `long long` for distances — road times can sum past `int` range — and apply the mod only to the *count*, never to the distances.
