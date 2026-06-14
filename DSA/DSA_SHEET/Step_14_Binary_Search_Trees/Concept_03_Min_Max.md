# Find Min / Max in a BST  🟢

**Reference:** [GeeksforGeeks — Find min and max in BST](https://www.geeksforgeeks.org/find-the-minimum-element-in-a-binary-search-tree/)

---

## Problem
Return the minimum and maximum values stored in a non-empty BST.

```
        8
       / \
      4   12
     /      \
    2        14

min = 2   max = 14
```

---

## Intuition
By the BST property the **smallest** value is the leftmost node (keep going left until there is no left child) and the **largest** value is the rightmost node (keep going right). No comparisons needed beyond following pointers.

---

## Code (C++)
```cpp
int findMin(TreeNode* root) {
    if (!root) return INT_MAX;          // empty tree guard
    while (root->left) root = root->left;   // leftmost node
    return root->val;
}

int findMax(TreeNode* root) {
    if (!root) return INT_MIN;          // empty tree guard
    while (root->right) root = root->right; // rightmost node
    return root->val;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — walk one root-to-leaf path |
| Space | O(1) iterative |

> ⚠️ Always guard against an empty tree (`root == nullptr`) before dereferencing — a common interview crash.
