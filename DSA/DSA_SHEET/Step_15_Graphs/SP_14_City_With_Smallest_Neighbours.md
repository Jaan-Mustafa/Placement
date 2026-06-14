# Find the City With Smallest Number of Neighbours  🔴

**Reference:** [LeetCode 1334 — Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)

---

## Problem
Given `n` cities, weighted bidirectional `edges[i] = [u, v, w]`, and a `distanceThreshold`, find the city that can reach the **fewest** other cities within that threshold. On ties, return the city with the **largest index**.

```
Input: n=4, edges=[[0,1,3],[1,2,1],[1,3,4],[2,3,1]], distanceThreshold=4
Output: 3   (city 3 reaches {0?no, 1:yes, 2:yes} -> 2 cities; tie, pick largest)
```

---

## Intuition
We need all-pairs shortest paths, so **Floyd-Warshall** is the clean fit. After filling the distance matrix, for each city count how many others are within `distanceThreshold`. We scan cities and keep the one with the smallest count; because we iterate upward and use `<=` on the count comparison, the **largest index** naturally wins ties.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
    const int INF = 1e9;
    vector<vector<int>> dist(n, vector<int>(n, INF));
    for (int i = 0; i < n; ++i) dist[i][i] = 0;

    for (auto& e : edges) {                 // undirected edges
        dist[e[0]][e[1]] = e[2];
        dist[e[1]][e[0]] = e[2];
    }

    // Floyd-Warshall all-pairs shortest paths
    for (int k = 0; k < n; ++k)
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                if (dist[i][k] < INF && dist[k][j] < INF)
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);

    int bestCity = 0, fewest = INT_MAX;
    for (int i = 0; i < n; ++i) {
        int count = 0;
        for (int j = 0; j < n; ++j)
            if (i != j && dist[i][j] <= distanceThreshold) ++count;
        // <= keeps the largest index on ties (i increases)
        if (count <= fewest) {
            fewest = count;
            bestCity = i;
        }
    }
    return bestCity;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n³) |
| Space | O(n²) |

> The tie-break (largest index wins) falls out for free by using `<=` while iterating cities in increasing order — using `<` would return the smallest index instead.
