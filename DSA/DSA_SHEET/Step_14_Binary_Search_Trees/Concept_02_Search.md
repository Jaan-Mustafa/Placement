# Search in a BST  🟢

**Reference:** [LeetCode 700 — Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/)

---

## Problem
Given the root of a BST and a value `target`, return the subtree rooted at the node whose value equals `target`, or `nullptr` if absent.

```
Tree: [4,2,7,1,3], target = 2   →   subtree  2
                                            / \
                                           1   3
```

---

## Intuition
Use the BST property: at each node, if `target` equals the node value we're done; if `target` is smaller go **left**, otherwise go **right**. Each comparison halves the search space, so no backtracking is ever needed — an iterative loop suffices and uses O(1) space.

---

## Code (C++)
```cpp
TreeNode* searchBST(TreeNode* root, int target) {
    while (root && root->val != target) {
        // target smaller → left subtree, else → right subtree
        root = (target < root->val) ? root->left : root->right;
    }
    return root;   // node with target, or nullptr if loop fell off the tree
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — H = tree height (O(log N) balanced, O(N) skewed) |
| Space | O(1) iterative |

> The recursive version costs O(H) stack space; the iterative loop above is strictly better with O(1) space.
