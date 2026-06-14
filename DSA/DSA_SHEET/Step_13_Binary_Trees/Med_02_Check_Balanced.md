# Check if Balanced  🟡

**Reference:** [LeetCode 110 — Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)

---

## Problem
A tree is **height-balanced** if for every node the heights of its two subtrees differ by at most 1. Return whether the tree is balanced.

```
        1               1
       / \               \
      2   3      vs        2     -> right is NOT balanced
     /                      \
    4                        3
```

---

## Intuition
Naively computing height at each node is O(N²). Instead, fold the balance check **into** the height computation: return the subtree height, but propagate a sentinel `-1` the moment any subtree is unbalanced, short-circuiting upward.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
int check(TreeNode* node) {
    if (!node) return 0;
    int lh = check(node->left);
    if (lh == -1) return -1;               // left already unbalanced
    int rh = check(node->right);
    if (rh == -1) return -1;               // right already unbalanced
    if (abs(lh - rh) > 1) return -1;       // this node unbalanced
    return 1 + max(lh, rh);                // otherwise return height
}

bool isBalanced(TreeNode* root) {
    return check(root) != -1;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — single postorder pass |
| Space | O(H) recursion stack |

> Using `-1` as an "unbalanced" sentinel avoids the O(N²) trap of recomputing heights at every node.
