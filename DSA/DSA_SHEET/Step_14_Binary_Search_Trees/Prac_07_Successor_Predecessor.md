# Inorder Successor / Predecessor in BST  🟡

**Reference:** [LeetCode 285 — Inorder Successor in BST](https://leetcode.com/problems/inorder-successor-in-bst/) · [GeeksforGeeks — Inorder predecessor and successor](https://www.geeksforgeeks.org/inorder-predecessor-successor-given-key-bst/)

---

## Problem
Given a BST and a value `key`, find its **inorder successor** (smallest value strictly greater than `key`) and **inorder predecessor** (largest value strictly smaller than `key`). Return `-1` if none.

```
Tree: [20,8,22,4,12,null,null,null,null,10,14], key = 8
successor   = 10      predecessor = 4
```

---

## Intuition
For the **successor**: walk down; whenever `node->val > key`, that node is a candidate successor — record it and go **left** to find something smaller-yet-still-greater; otherwise go **right**. The **predecessor** is the mirror: when `node->val < key`, record it and go **right**, else go **left**. This is the same candidate-tracking pattern as floor/ceil but with strict inequalities.

---

## Code (C++)
```cpp
int inorderSuccessor(TreeNode* root, int key) {
    int succ = -1;
    while (root) {
        if (root->val > key) { succ = root->val; root = root->left; }
        else                 { root = root->right; }   // val <= key → go right
    }
    return succ;
}

int inorderPredecessor(TreeNode* root, int key) {
    int pred = -1;
    while (root) {
        if (root->val < key) { pred = root->val; root = root->right; }
        else                 { root = root->left; }    // val >= key → go left
    }
    return pred;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — single descent each |
| Space | O(1) iterative |

> ⚠️ Successor/predecessor here are **strict** (≠ key), unlike floor/ceil which are inclusive (≤/≥). The only code change is using `>`/`<` instead of `>=`/`<=`.
