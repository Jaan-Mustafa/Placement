# Network Delay Time  🟡

**Reference:** [LeetCode 743 — Network Delay Time](https://leetcode.com/problems/network-delay-time/)

---

## Problem
A signal starts at node `k` in a directed weighted network of `n` nodes (`times[i] = [u, v, w]`). Return the time for **all** nodes to receive the signal, or `-1` if some node never does.

```
Input: times=[[2,1,1],[2,3,1],[3,4,1]], n=4, k=2
Output: 2   (node 4 is reached last, at time 2)
```

---

## Intuition
The time for all nodes to receive the signal is the **maximum shortest-path distance** from the source `k`. So run Dijkstra from `k`, then take the largest finalised distance. If any node is still at infinity it's unreachable, so return `-1`.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    // 1-indexed nodes 1..n; adjacency list {neighbour, weight}
    vector<vector<pair<int,int>>> adj(n + 1);
    for (auto& t : times) adj[t[0]].push_back({t[1], t[2]});

    vector<int> dist(n + 1, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    dist[k] = 0;
    pq.push({0, k});

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (d > dist[node]) continue;          // stale
        for (auto& [nbr, w] : adj[node])
            if (d + w < dist[nbr]) {
                dist[nbr] = d + w;
                pq.push({dist[nbr], nbr});
            }
    }

    int ans = 0;
    for (int i = 1; i <= n; ++i) {
        if (dist[i] == INT_MAX) return -1;     // some node unreachable
        ans = max(ans, dist[i]);               // slowest node defines the time
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(E log V) |
| Space | O(V + E) |

> The answer is the *max* over all shortest distances, not the sum — the signal propagates in parallel, so total time is bounded by the farthest node.
