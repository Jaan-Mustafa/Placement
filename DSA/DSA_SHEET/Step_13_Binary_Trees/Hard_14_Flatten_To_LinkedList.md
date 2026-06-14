# Flatten Binary Tree to Linked List  🔴

**Reference:** [LeetCode 114 — Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)

---

## Problem
Flatten the tree in-place into a "linked list" following **preorder**, using the `right` pointers (all `left` pointers become null).

```
        1               1
       / \               \
      2   5      ->        2
     / \   \                \
    3   4   6                3
                              \
                               4 ...
```

---

## Intuition
Use a Morris-style rewiring. For each node with a left child, find the rightmost node of that left subtree, attach the current right subtree under it, then move the left subtree to the right and null the left pointer. Advance to the right.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void flatten(TreeNode* root) {
    TreeNode* cur = root;
    while (cur) {
        if (cur->left) {
            TreeNode* pred = cur->left;        // rightmost of left subtree
            while (pred->right) pred = pred->right;
            pred->right = cur->right;          // stitch right subtree after it
            cur->right  = cur->left;           // move left subtree to right
            cur->left   = nullptr;             // clear left pointer
        }
        cur = cur->right;                      // proceed down the new chain
    }
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) — in-place, no recursion or stack |

> Order matters: stitch the right subtree onto the left subtree's rightmost node *before* overwriting `cur->right`, or you lose the right subtree.
