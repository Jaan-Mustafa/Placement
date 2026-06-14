# Vertical Order Traversal  🔴

**Reference:** [LeetCode 987 — Vertical Order Traversal of a Binary Tree](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/)

---

## Problem
Group nodes by their **vertical column** (root = column 0, left = −1, right = +1). Within a column, order by row (depth); ties at the same (row, col) are sorted by value.

```
        1
       / \
      2   3        Columns: -1:[2]  0:[1]  +1:[3]  ->  [[2],[1],[3]]
```

---

## Intuition
BFS/DFS tracking `(col, row)` for each node. Use a sorted map keyed by `col`, and within each column a sorted map keyed by `row`, holding a **multiset** of values so same-cell ties break by value.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<vector<int>> verticalTraversal(TreeNode* root) {
    // col -> (row -> multiset<value>)
    map<int, map<int, multiset<int>>> nodes;
    queue<tuple<TreeNode*, int, int>> q;     // {node, col, row}
    q.push({root, 0, 0});
    while (!q.empty()) {
        auto [node, col, row] = q.front(); q.pop();
        nodes[col][row].insert(node->val);
        if (node->left)  q.push({node->left,  col - 1, row + 1});
        if (node->right) q.push({node->right, col + 1, row + 1});
    }
    vector<vector<int>> res;
    for (auto& [col, rows] : nodes) {
        vector<int> column;
        for (auto& [row, vals] : rows)
            column.insert(column.end(), vals.begin(), vals.end());
        res.push_back(column);
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) — map/multiset ordering |
| Space | O(N) |

> The same-(row,col) tie-break by value is what makes LC 987 differ from a plain vertical traversal — a `multiset` handles it cleanly.
