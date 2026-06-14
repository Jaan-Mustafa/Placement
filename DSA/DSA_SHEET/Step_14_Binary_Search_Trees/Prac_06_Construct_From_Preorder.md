# Construct BST from Preorder Traversal  🟡

**Reference:** [LeetCode 1008 — Construct Binary Search Tree from Preorder Traversal](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/)

---

## Problem
Given the **preorder** traversal of a BST, rebuild the tree and return its root.

```
preorder = [8,5,1,7,10,12]

        8
       / \
      5   10
     / \    \
    1   7    12
```

---

## Intuition
In preorder the **first** element is always the root. The next values that are **smaller** than the root form the left subtree, and the rest form the right subtree. Rather than partition the array repeatedly (O(N²)), use an **upper bound**: scan the preorder once, building each node only while the current value is below the allowed bound. A node's left subtree is bounded by the node value; its right subtree inherits the parent's bound.

---

## Code (C++)
```cpp
TreeNode* build(vector<int>& pre, int bound, int& i) {
    // stop if exhausted or current value belongs to an ancestor's right side
    if (i == (int)pre.size() || pre[i] > bound) return nullptr;
    TreeNode* root = new TreeNode(pre[i++]);     // consume current as root
    root->left  = build(pre, root->val, i);      // left: values < root->val
    root->right = build(pre, bound, i);          // right: values < parent bound
    return root;
}

TreeNode* bstFromPreorder(vector<int>& preorder) {
    int i = 0;
    return build(preorder, INT_MAX, i);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each element consumed exactly once |
| Space | O(H) recursion stack |

> The shared index `i` (passed by reference) is what makes this single-pass; sorting preorder gives inorder, so you could also use the (inorder, preorder) build, but the bound trick is faster and needs no extra array.
