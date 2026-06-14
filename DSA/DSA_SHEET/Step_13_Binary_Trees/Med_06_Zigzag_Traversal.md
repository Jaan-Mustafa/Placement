# Zigzag / Spiral Traversal  🟡

**Reference:** [LeetCode 103 — Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)

---

## Problem
Level order, but alternate direction each level: left→right, then right→left, etc.

```
        1
       / \
      2   3        Output: [[1], [3,2], [4,5,6,7]]
     / \ / \
    4  5 6  7
```

---

## Intuition
Standard BFS by levels, but track a `leftToRight` flag. When filling a level right-to-left, write each popped node into its mirrored index instead of appending.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    bool leftToRight = true;
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            int idx = leftToRight ? i : (sz - 1 - i);  // mirror when reversed
            level[idx] = node->val;
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        leftToRight = !leftToRight;        // flip direction each level
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
| Space | O(N) — queue + output |

> Children are always enqueued left-then-right; only the *write index* flips, so the BFS itself stays unchanged.
