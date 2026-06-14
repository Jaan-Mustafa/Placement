# Top View of Binary Tree  🟡

**Reference:** [GeeksforGeeks — Top View of Binary Tree](https://www.geeksforgeeks.org/print-nodes-top-view-binary-tree/)

---

## Problem
Print nodes visible when the tree is viewed from directly above — the first node encountered in each vertical column.

```
        1
       / \
      2   3        Top view: 4 2 1 3 7
     / \ / \
    4  5 6  7
```

---

## Intuition
BFS tracking horizontal distance (`hd`). For each column store **only the first** node seen (BFS guarantees it's the topmost). A sorted map by `hd` gives left-to-right output.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> topView(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    map<int, int> top;                       // hd -> first node value
    queue<pair<TreeNode*, int>> q;           // {node, hd}
    q.push({root, 0});
    while (!q.empty()) {
        auto [node, hd] = q.front(); q.pop();
        if (top.find(hd) == top.end())       // first node at this column
            top[hd] = node->val;
        if (node->left)  q.push({node->left,  hd - 1});
        if (node->right) q.push({node->right, hd + 1});
    }
    for (auto& [hd, val] : top) res.push_back(val);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) — map insertions |
| Space | O(N) |

> Must use **BFS**, not DFS: only level-by-level order guarantees the first node recorded per column is actually the highest one.
