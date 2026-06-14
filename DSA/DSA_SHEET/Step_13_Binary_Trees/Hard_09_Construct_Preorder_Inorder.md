# Construct Tree from Preorder & Inorder  🔴

**Reference:** [LeetCode 105 — Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

---

## Problem
Rebuild the unique binary tree from its preorder and inorder traversals.

```
preorder = [3,9,20,15,7]              3
inorder  = [9,3,15,20,7]    ->       / \
                                     9  20
                                       /  \
                                      15   7
```

---

## Intuition
The first element of preorder is the root. Its index in inorder splits inorder into the left subtree (before it) and right subtree (after it). The left subtree's size tells us how much of preorder belongs to it. Recurse. A hash map of value→inorder-index makes the split O(1).

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
TreeNode* build(vector<int>& pre, int preL, int preR,
                vector<int>& in, int inL, int inR,
                unordered_map<int,int>& idx) {
    if (preL > preR || inL > inR) return nullptr;
    int rootVal = pre[preL];
    TreeNode* root = new TreeNode(rootVal);
    int pos = idx[rootVal];                 // root position in inorder
    int leftSize = pos - inL;               // nodes in left subtree
    root->left  = build(pre, preL + 1, preL + leftSize,
                        in, inL, pos - 1, idx);
    root->right = build(pre, preL + leftSize + 1, preR,
                        in, pos + 1, inR, idx);
    return root;
}

TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    unordered_map<int,int> idx;
    for (int i = 0; i < (int)inorder.size(); i++) idx[inorder[i]] = i;
    return build(preorder, 0, preorder.size() - 1,
                 inorder, 0, inorder.size() - 1, idx);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) with the index map (O(N²) without) |
| Space | O(N) map + O(H) recursion |

> `leftSize = pos - inL` is the linchpin — it splits the preorder range correctly. Off-by-one here silently corrupts the whole tree.
