# Bottom View of Binary Tree  🟡

**Reference:** [GeeksforGeeks — Bottom View of a Binary Tree](https://www.geeksforgeeks.org/bottom-view-binary-tree/)

---

## Problem
Print nodes visible when the tree is viewed from directly below — the last node encountered in each vertical column.

```
        1
       / \
      2   3        Bottom view: 4 2 6 3 7
     / \ / \
    4  5 6  7
```

---

## Intuition
Same BFS as top view, but **overwrite** the value for each column every time — the last write in BFS order is the lowest node in that column.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> bottomView(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    map<int, int> bottom;                    // hd -> latest node value
    queue<pair<TreeNode*, int>> q;           // {node, hd}
    q.push({root, 0});
    while (!q.empty()) {
        auto [node, hd] = q.front(); q.pop();
        bottom[hd] = node->val;              // always overwrite
        if (node->left)  q.push({node->left,  hd - 1});
        if (node->right) q.push({node->right, hd + 1});
    }
    for (auto& [hd, val] : bottom) res.push_back(val);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(N) |

> The only difference from top view: drop the `find` guard and always overwrite. BFS ensures the final write per column is the bottom-most node.
