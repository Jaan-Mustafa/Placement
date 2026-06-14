# Kth Smallest / Largest Element in BST  🟡

**Reference:** [LeetCode 230 — Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

---

## Problem
Given a BST and integer `k`, return the k-th **smallest** value (1-indexed). The k-th largest is the symmetric reverse-inorder variant.

```
Tree: [3,1,4,null,2], k = 1   →   1
Tree: [5,3,6,2,4,null,null,1], k = 3   →   3
```

---

## Intuition
An **inorder** traversal of a BST visits values in ascending order. Count nodes as you visit them; the k-th visited node is the k-th smallest. Stop early once the count hits `k`. For the k-th **largest**, do a **reverse inorder** (right → node → left) which visits values in descending order.

---

## Code (C++)
```cpp
void inorder(TreeNode* root, int k, int& cnt, int& ans) {
    if (!root || cnt >= k) return;     // prune once we have the answer
    inorder(root->left, k, cnt, ans);
    if (++cnt == k) { ans = root->val; return; }   // k-th node visited
    inorder(root->right, k, cnt, ans);
}

int kthSmallest(TreeNode* root, int k) {
    int cnt = 0, ans = -1;
    inorder(root, k, cnt, ans);
    return ans;
}

// k-th largest: swap the two recursive calls (right before left).
int kthLargest(TreeNode* root, int k) {
    int cnt = 0, ans = -1;
    function<void(TreeNode*)> rev = [&](TreeNode* node) {
        if (!node || cnt >= k) return;
        rev(node->right);
        if (++cnt == k) { ans = node->val; return; }
        rev(node->left);
    };
    rev(root);
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H + k) — descend to leftmost, then visit k nodes |
| Space | O(H) recursion stack |

> If the tree is modified frequently and you query k often, augment each node with a subtree-size count to answer in O(H) directly.
