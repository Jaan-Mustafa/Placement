# Most Stones Removed With Same Row or Column  🔴

**Reference:** [LeetCode 947 — Most Stones Removed With Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/)

---

## Problem
Stones sit on a 2D plane. A stone can be removed if it shares its row or column with another remaining stone. Return the maximum number of stones that can be removed.

```
Input: stones=[[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]
Output: 5   (one stone must remain per connected component)
```

---

## Intuition
Two stones are "linked" if they share a row or a column; linked stones form a connected component. Within any component you can keep removing until exactly **one** stone is left (the last removal target). So the answer is `totalStones - numberOfComponents`. Use a Disjoint Set keyed by rows and columns. Rows and columns share the same integer space, so offset column indices by a large constant (e.g. `10001`) to keep them distinct. Union each stone's row with its column; the number of distinct roots among used row/col labels is the component count.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    DSU(int n): parent(n), rank_(n, 0) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }
    void unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return;
        if (rank_[a] < rank_[b]) swap(a, b);
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;
    }
};

int removeStones(vector<vector<int>>& stones) {
    const int OFFSET = 10001;            // columns live above this to avoid clashes
    DSU dsu(2 * OFFSET);
    unordered_set<int> used;             // track which row/col labels were touched

    for (auto& s : stones) {
        int row = s[0], col = s[1] + OFFSET;
        dsu.unite(row, col);             // link this stone's row and column
        used.insert(row);
        used.insert(col);
    }

    int components = 0;
    for (int label : used)
        if (dsu.find(label) == label) ++components;   // count distinct roots

    return (int)stones.size() - components;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N·α) |
| Space | O(N) |

> The trick is treating rows and columns as nodes (not the stones themselves) and offsetting columns; a stone is merely the edge that unites its row with its column.
