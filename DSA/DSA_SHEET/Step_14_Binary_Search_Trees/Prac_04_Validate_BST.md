# Check if a Tree is a BST  🟡

**Reference:** [LeetCode 98 — Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)

---

## Problem
Determine whether a binary tree is a valid BST: every node's left subtree holds only **smaller** values and its right subtree only **larger** values (strict, no duplicates).

```
    2          5
   / \        / \
  1   3  ✔   1   4   ✘  (4's left child 3 violates 5's range)
                / \
               3   6
```

---

## Intuition
A node being larger than its left child and smaller than its right child is **not** enough — every node must lie within a valid `(low, high)` range inherited from all its ancestors. Recurse with an open interval: going left tightens the upper bound to the node's value, going right tightens the lower bound.

---

## Code (C++)
```cpp
// use long long bounds so INT_MIN / INT_MAX node values don't false-fail
bool validate(TreeNode* node, long long low, long long high) {
    if (!node) return true;
    if (node->val <= low || node->val >= high) return false;   // out of range
    return validate(node->left,  low, node->val)      // left: cap upper bound
        && validate(node->right, node->val, high);    // right: raise lower bound
}

bool isValidBST(TreeNode* root) {
    return validate(root, LLONG_MIN, LLONG_MAX);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — visit every node once |
| Space | O(H) recursion stack |

> ⚠️ The classic bug is comparing only parent-child. The `(low, high)` range propagated from **all** ancestors is what makes it correct. Use `long long` bounds to handle node values of `INT_MIN`/`INT_MAX`.
