# Disjoint Set (Union by Rank & Path Compression)  🟡

**Reference:** [GeeksforGeeks — Introduction to Disjoint Set (Union-Find)](https://www.geeksforgeeks.org/introduction-to-disjoint-set-data-structure-or-union-find-algorithm/)

---

## Problem
Maintain a collection of disjoint sets supporting two near-constant-time operations: `find(x)` returns the representative of `x`'s set, and `unite(a, b)` merges the sets containing `a` and `b`.

```
Input: n=5; unite(0,1), unite(1,2), unite(3,4)
Query: find(0)==find(2)? -> true     find(0)==find(4)? -> false
Output: components = {0,1,2}, {3,4}
```

---

## Intuition
Each element points to a parent; following parents up reaches the set's root representative. Two optimizations make this almost O(1) amortized. **Path compression**: during `find`, re-point every visited node directly to the root so future queries are flat. **Union by rank**: when merging, attach the shorter tree under the taller one, keeping trees shallow and bounding height growth. Together they yield inverse-Ackermann α(n) time per operation — effectively constant.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    DSU(int n): parent(n), rank_(n, 0) {
        iota(parent.begin(), parent.end(), 0);   // each node is its own root
    }
    int find(int x) {                             // path compression
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }
    void unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return;                       // already same set
        if (rank_[a] < rank_[b]) swap(a, b);      // attach shorter under taller
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;     // tie => height grows by 1
    }
};

int main() {
    DSU dsu(5);
    dsu.unite(0, 1); dsu.unite(1, 2); dsu.unite(3, 4);
    cout << (dsu.find(0) == dsu.find(2)) << "\n";  // 1 (same set)
    cout << (dsu.find(0) == dsu.find(4)) << "\n";  // 0 (different sets)
    return 0;
}
```

---

## Complexity
| | |
|---|---|
| Time (per op) | O(α(n)) ≈ O(1) amortized |
| Space | O(n) |

> Path compression alone or union by rank alone gives O(log n); you need BOTH together to reach the near-constant inverse-Ackermann bound.
