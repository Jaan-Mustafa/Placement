# Preorder Traversal (Recursive)  🟢

**Reference:** [LeetCode 144 — Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

---

## Problem
Visit nodes in **Root → Left → Right** order.

```
        1
       / \
      2   3        Preorder: 1 2 4 5 3
     / \
    4   5
```

---

## Intuition
Process the current node first, then recurse into the left subtree, then the right subtree. The recursion stack mirrors the path from root to the current node.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void preorder(TreeNode* node, vector<int>& out) {
    if (!node) return;            // base case: empty subtree
    out.push_back(node->val);     // Root
    preorder(node->left,  out);   // Left
    preorder(node->right, out);   // Right
}

vector<int> preorderTraversal(TreeNode* root) {
    vector<int> out;
    preorder(root, out);
    return out;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each node visited once |
| Space | O(H) recursion stack (H = height; O(N) worst, O(log N) balanced) |

> Order mnemonic: **N**ode-**L**eft-**R**ight. The node is emitted *before* descending.
