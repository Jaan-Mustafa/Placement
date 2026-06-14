# Construct Tree from Postorder & Inorder  🔴

**Reference:** [LeetCode 106 — Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

---

## Problem
Rebuild the unique binary tree from its inorder and postorder traversals.

```
inorder   = [9,3,15,20,7]             3
postorder = [9,15,7,20,3]   ->       / \
                                     9  20
                                       /  \
                                      15   7
```

---

## Intuition
The **last** element of postorder is the root. Locate it in inorder to split left/right subtrees. Process postorder from the **right end**: build the right subtree first, then the left, since postorder ends with root then right-subtree-last.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
TreeNode* build(vector<int>& post, int postL, int postR,
                vector<int>& in, int inL, int inR,
                unordered_map<int,int>& idx) {
    if (postL > postR || inL > inR) return nullptr;
    int rootVal = post[postR];               // last of postorder = root
    TreeNode* root = new TreeNode(rootVal);
    int pos = idx[rootVal];
    int leftSize = pos - inL;
    root->left  = build(post, postL, postL + leftSize - 1,
                        in, inL, pos - 1, idx);
    root->right = build(post, postL + leftSize, postR - 1,
                        in, pos + 1, inR, idx);
    return root;
}

TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
    unordered_map<int,int> idx;
    for (int i = 0; i < (int)inorder.size(); i++) idx[inorder[i]] = i;
    return build(postorder, 0, postorder.size() - 1,
                 inorder, 0, inorder.size() - 1, idx);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) with the index map |
| Space | O(N) map + O(H) recursion |

> Root is the **last** element of postorder here (vs first of preorder). The postorder range split uses `leftSize` exactly like the preorder version.
