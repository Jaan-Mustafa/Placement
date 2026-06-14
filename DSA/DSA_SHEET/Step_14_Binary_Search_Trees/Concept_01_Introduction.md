# Introduction to BST  🟢

**Reference:** [GeeksforGeeks — Binary Search Tree](https://www.geeksforgeeks.org/binary-search-tree-data-structure/) · [LeetCode 700 — Search in a BST](https://leetcode.com/problems/search-in-a-binary-search-tree/)

---

## Problem
Understand the **Binary Search Tree (BST)** property and why it makes search/insert/delete fast.

A BST is a binary tree where, for **every** node:
- all values in its **left** subtree are **smaller** than the node, and
- all values in its **right** subtree are **larger** than the node.

```
        8
       / \
      4   12
     / \    \
    2   6    14
```
An **inorder traversal** of a BST always yields values in **sorted ascending order**: `2 4 6 8 12 14`.

---

## Intuition
The ordering invariant lets you discard half the tree at every step (like binary search on an array). At each node you compare the target with `node->val`: go left if smaller, right if larger. On a **balanced** BST this gives O(log N) operations; on a **skewed** BST (insertions in sorted order) it degrades to O(N) — a straight line.

---

## Code (C++)
```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// Verify the inorder sequence is strictly increasing — the defining BST test.
void inorder(TreeNode* root, vector<int>& out) {
    if (!root) return;
    inorder(root->left, out);   // left subtree (all smaller)
    out.push_back(root->val);   // node
    inorder(root->right, out);  // right subtree (all larger)
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) balanced, O(N) skewed — per search/insert/delete |
| Space | O(H) recursion stack, H = height |

> ⚠️ A plain BST is **not** self-balancing. A run of sorted inserts produces a degenerate O(N) tree — real systems use AVL / Red-Black trees to keep height O(log N).
