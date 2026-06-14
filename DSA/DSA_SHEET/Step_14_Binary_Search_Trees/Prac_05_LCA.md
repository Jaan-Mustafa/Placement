# Lowest Common Ancestor in a BST  🟡

**Reference:** [LeetCode 235 — Lowest Common Ancestor of a BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

---

## Problem
Given a BST and two nodes `p` and `q` (both present), return their lowest common ancestor — the deepest node that has both as descendants (a node may be a descendant of itself).

```
Tree: [6,2,8,0,4,7,9,null,null,3,5]
LCA(2, 8) = 6      LCA(2, 4) = 2
```

---

## Intuition
Exploit the BST ordering. Starting at the root: if **both** `p` and `q` are smaller than the current node, the LCA must be in the **left** subtree; if both are larger, it's in the **right** subtree. The moment they **split** (one ≤ node ≤ other, or the node equals one of them), the current node is the LCA — it's the first place the paths to `p` and `q` diverge.

---

## Code (C++)
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val)
            root = root->left;            // both in left subtree
        else if (p->val > root->val && q->val > root->val)
            root = root->right;           // both in right subtree
        else
            return root;                  // split point (or node == p/q) → LCA
    }
    return nullptr;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — single descent, no backtracking |
| Space | O(1) iterative |

> This is much cheaper than the generic binary-tree LCA (O(N)); the BST ordering lets you walk straight to the split point.
