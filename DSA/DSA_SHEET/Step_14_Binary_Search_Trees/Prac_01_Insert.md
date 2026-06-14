# Insert a Node in BST  🟡

**Reference:** [LeetCode 701 — Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/)

---

## Problem
Insert a value into a BST and return the new root. The value is guaranteed not to already exist. Any valid BST result is accepted.

```
Tree: [4,2,7,1,3], insert 5

        4                 4
       / \               / \
      2   7      →       2   7
     / \               / \  /
    1   3             1  3 5
```

---

## Intuition
Walk down the tree as if searching for the value. Whenever the child you want to descend into is `nullptr`, that empty slot is exactly where the new node belongs — attach it there as a leaf. The BST stays valid because we followed the ordering all the way down.

---

## Code (C++)
```cpp
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);    // empty tree → new node is root
    TreeNode* cur = root;
    while (true) {
        if (val < cur->val) {
            if (!cur->left) { cur->left = new TreeNode(val); break; }
            cur = cur->left;
        } else {
            if (!cur->right) { cur->right = new TreeNode(val); break; }
            cur = cur->right;
        }
    }
    return root;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — descend one path to a leaf |
| Space | O(1) iterative |

> A new value is always inserted as a **leaf** in a plain BST; the existing structure is never reorganized.
