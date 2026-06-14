# Level Order Traversal  🟢

**Reference:** [LeetCode 102 — Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

---

## Problem
Return node values level by level, top to bottom, left to right.

```
        1
       / \
      2   3        Output: [[1], [2,3], [4,5]]
     / \
    4   5
```

---

## Intuition
Breadth-first search using a **queue**. Push the root, then repeatedly drain the queue one *level* at a time: record the current queue size, pop exactly that many nodes into the current level's list, and enqueue their children.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();            // nodes in this level
        vector<int> level;
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) — queue holds up to a full level (~N/2 nodes) |

> Capturing `q.size()` *before* the inner loop is what cleanly separates levels — don't read it inside.
