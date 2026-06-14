# Right / Left View of Binary Tree  🟡

**Reference:** [LeetCode 199 — Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

---

## Problem
The **right view** is the set of nodes visible from the right side — the last node of each level. The **left view** is symmetric (first node of each level).

```
        1
       / \
      2   3        Right view: 1 3 7
     / \   \
    4   5   7      Left view:  1 2 4
```

---

## Intuition
DFS visiting **Root → Right → Left**. The first node reached at each new depth is the rightmost. Track results indexed by depth; push only when the current depth equals the result size. For the left view, just visit left before right.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void dfs(TreeNode* node, int depth, vector<int>& res) {
    if (!node) return;
    if (depth == (int)res.size())            // first node seen at this depth
        res.push_back(node->val);
    dfs(node->right, depth + 1, res);        // right first -> right view
    dfs(node->left,  depth + 1, res);
}

vector<int> rightSideView(TreeNode* root) {
    vector<int> res;
    dfs(root, 0, res);
    return res;
}
// For LEFT view: swap the two dfs() calls (visit left before right).
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> `depth == res.size()` is the trick — it captures exactly the first node of each new level, no level-size bookkeeping needed.
