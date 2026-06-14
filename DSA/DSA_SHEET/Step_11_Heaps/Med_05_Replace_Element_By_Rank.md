# Replace Each Element by Its Rank  🟡

**Reference:** [GeeksforGeeks — Replace each element by its rank in the array](https://www.geeksforgeeks.org/replace-each-element-by-its-rank-in-the-given-array/)

---

## Problem
Replace every element by its **rank**: the smallest distinct value has rank 1, the next distinct value rank 2, and so on (equal values share a rank).

```
Input:  [20, 15, 26, 2, 98, 6]   Output: [4, 3, 5, 1, 6, 2]
Input:  [2, 2, 1, 6]             Output: [2, 2, 1, 3]
```

---

## Intuition
Rank = position among **sorted distinct values**. A min-heap gives values in ascending order; pop them, assigning increasing ranks while skipping duplicates, and record each value's rank in a map. Then map every original element to its rank. (A plain sort + map is equivalent; the heap variant illustrates the ordered-extraction pattern.)

---

## Code (C++)
```cpp
vector<int> replaceByRank(vector<int>& arr) {
    priority_queue<int, vector<int>, greater<int>> mn(arr.begin(), arr.end());
    unordered_map<int,int> rank;       // value -> rank
    int r = 0, prev = INT_MIN;
    while (!mn.empty()) {
        int v = mn.top(); mn.pop();
        if (v != prev) { r++; prev = v; }  // new distinct value -> next rank
        rank[v] = r;                       // equal values keep same rank
    }
    vector<int> res;
    res.reserve(arr.size());
    for (int x : arr) res.push_back(rank[x]);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(N) |

> ⚠️ Duplicates must share a rank — guard the increment with `v != prev`. Initialize `prev = INT_MIN` so the first popped value always triggers rank 1.
