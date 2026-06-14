# Symmetric Binary Tree  🟡

**Reference:** [LeetCode 101 — Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)

---

## Problem
Return true if the tree is a mirror of itself around its center.

```
        1                1
       / \              / \
      2   2            2   2
     / \ / \            \   \
    3  4 4  3   YES      3   3   NO
```

---

## Intuition
A tree is symmetric iff its left subtree is the **mirror** of its right subtree. Two subtrees mirror each other when their roots are equal and `left.left` mirrors `right.right` while `left.right` mirrors `right.left` (cross comparison).

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
bool mirror(TreeNode* a, TreeNode* b) {
    if (!a && !b) return true;             // both empty
    if (!a || !b) return false;            // one empty
    if (a->val != b->val) return false;    // values differ
    return mirror(a->left,  b->right)      // outer pair
        && mirror(a->right, b->left);      // inner pair (crossed)
}

bool isSymmetric(TreeNode* root) {
    if (!root) return true;
    return mirror(root->left, root->right);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> The crossed recursion `(a->left, b->right)` and `(a->right, b->left)` is the whole trick — comparing same-side children instead checks for *identical*, not *mirror*.
