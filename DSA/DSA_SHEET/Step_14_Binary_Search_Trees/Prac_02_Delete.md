# Delete a Node in BST  🟡

**Reference:** [LeetCode 450 — Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)

---

## Problem
Delete the node with a given key from a BST and return the new root, keeping the BST valid.

```
Tree: [5,3,6,2,4,null,7], delete 3

        5                    5
       / \                  / \
      3   6        →        4   6
     / \   \              /      \
    2   4   7            2        7
```

---

## Intuition
Search for the node. Once found there are three cases:
1. **No left child** → replace the node with its right child.
2. **No right child** → replace the node with its left child.
3. **Two children** → find the **inorder successor** (smallest in the right subtree), copy its value into the node, then recursively delete that successor (which has at most one child). This preserves the ordering.

---

## Code (C++)
```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val)      root->left  = deleteNode(root->left, key);
    else if (key > root->val) root->right = deleteNode(root->right, key);
    else {
        // found the node to delete
        if (!root->left)  { TreeNode* r = root->right; delete root; return r; }
        if (!root->right) { TreeNode* l = root->left;  delete root; return l; }
        // two children: grab inorder successor (min of right subtree)
        TreeNode* succ = root->right;
        while (succ->left) succ = succ->left;
        root->val = succ->val;                       // copy value up
        root->right = deleteNode(root->right, succ->val);  // remove successor
    }
    return root;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — search path plus successor search, both bounded by height |
| Space | O(H) recursion stack |

> ⚠️ The two-child case is the tricky one — you can use either the inorder successor (min of right) or inorder predecessor (max of left); just be consistent and recurse to actually free that helper node.
