# Inorder Traversal (Recursive)  🟢

**Reference:** [LeetCode 94 — Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

---

## Problem
Visit nodes in **Left → Root → Right** order.

```
        1
       / \
      2   3        Inorder: 4 2 5 1 3
     / \
    4   5
```

---

## Intuition
Recurse fully into the left subtree, emit the node, then recurse into the right subtree. For a **Binary Search Tree** this yields values in sorted order — a frequently used property.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void inorder(TreeNode* node, vector<int>& out) {
    if (!node) return;            // base case
    inorder(node->left,  out);    // Left
    out.push_back(node->val);     // Root
    inorder(node->right, out);    // Right
}

vector<int> inorderTraversal(TreeNode* root) {
    vector<int> out;
    inorder(root, out);
    return out;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> On a BST, inorder traversal is sorted ascending — remember this for validation problems.
