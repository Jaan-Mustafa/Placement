# Lowest Common Ancestor (LCA)  🟡

**Reference:** [LeetCode 236 — Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

---

## Problem
Find the lowest node that has both `p` and `q` as descendants (a node can be its own ancestor).

```
        3
       / \
      5   1         LCA(5, 1) = 3
     / \           LCA(5, 4) = 5
    6   2
       / \
      7   4
```

---

## Intuition
Postorder recursion. If the current node is null or equals `p` or `q`, return it. Recurse both sides: if both return non-null, the current node is the split point → it's the LCA. Otherwise propagate whichever side is non-null.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;  // base / found
    TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left && right) return root;     // p and q split here -> LCA
    return left ? left : right;         // propagate the found side
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> This assumes both `p` and `q` exist in the tree (LC 236 guarantees it). The "found on both sides" condition is what pins down the lowest ancestor.
