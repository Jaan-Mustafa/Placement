# Largest BST in a Binary Tree  🔴

**Reference:** [GeeksforGeeks — Largest BST in a Binary Tree](https://www.geeksforgeeks.org/largest-bst-binary-tree-set-2/) · [LeetCode 333 — Largest BST Subtree](https://leetcode.com/problems/largest-bst-subtree/)

---

## Problem
Given a binary tree (not necessarily a BST), find the size (number of nodes) of the **largest subtree that is itself a valid BST**.

```
    10
   /  \
  5    15
 / \     \
1   8     7

Largest BST subtree = the one rooted at 5: {1,5,8} → size 3
(the whole tree is not a BST: 7 < 15 violates)
```

---

## Intuition
Process the tree **bottom-up** (postorder). For each node compute, from its children's results, three things: the subtree's `min`, `max`, and BST `size`. A node forms a valid BST only when **both** children are valid BSTs **and** `leftMax < node->val < rightMin`. If valid, its size is `leftSize + rightSize + 1` and its range merges the children's; if invalid, mark size as `-1` (or `INT_MIN`) so any ancestor using it is also invalidated. Track the global maximum size seen.

---

## Code (C++)
```cpp
struct Info {
    int minV, maxV, size;   // size == -1 means "subtree is not a BST"
    bool isBST;
};

Info solve(TreeNode* root, int& best) {
    if (!root) return {INT_MAX, INT_MIN, 0, true};   // empty is a BST of size 0
    Info l = solve(root->left, best);
    Info r = solve(root->right, best);
    if (l.isBST && r.isBST && l.maxV < root->val && root->val < r.minV) {
        int sz = l.size + r.size + 1;
        best = max(best, sz);
        return {min(l.minV, root->val), max(r.maxV, root->val), sz, true};
    }
    return {INT_MIN, INT_MAX, -1, false};   // poison values invalidate ancestors
}

int largestBST(TreeNode* root) {
    int best = 0;
    solve(root, best);
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each node visited once, O(1) merge work per node |
| Space | O(H) recursion stack |

> ⚠️ The crucial trick is the **poison range** for an invalid subtree (`min = INT_MIN`, `max = INT_MAX`): it guarantees no ancestor can ever satisfy `leftMax < val < rightMin`, so invalidity propagates upward correctly. Empty subtrees use the opposite extremes so they never break a parent's check.
