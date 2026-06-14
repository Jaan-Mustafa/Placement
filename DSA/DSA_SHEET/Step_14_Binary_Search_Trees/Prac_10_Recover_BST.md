# Recover BST (Fix Two Swapped Nodes)  🔴

**Reference:** [LeetCode 99 — Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/)

---

## Problem
Exactly two nodes of a BST were swapped by mistake. Recover the tree **without changing its structure** (swap the two values back).

```
    3                  2
   / \                / \
  1   4      →       1   4
     /                  /
    2                  3
(3 and 2 were swapped)
```

---

## Intuition
A correct BST has a strictly increasing inorder sequence. Swapping two nodes creates either one or two descending "dips". Track the previous node during inorder:
- **First** violation (`prev->val > cur->val`): mark `first = prev` and `middle = cur`.
- **Second** violation: mark `last = cur`.

If the two swapped nodes are **adjacent** in inorder there's only one dip → swap `first` and `middle`. If they're **non-adjacent** there are two dips → swap `first` and `last`.

---

## Code (C++)
```cpp
class Solution {
    TreeNode *first = nullptr, *middle = nullptr, *last = nullptr, *prev = nullptr;
    void inorder(TreeNode* root) {
        if (!root) return;
        inorder(root->left);
        if (prev && root->val < prev->val) {     // a descending dip
            if (!first) { first = prev; middle = root; }  // 1st violation
            else        { last = root; }                  // 2nd violation
        }
        prev = root;
        inorder(root->right);
    }
public:
    void recoverTree(TreeNode* root) {
        inorder(root);
        if (first && last) swap(first->val, last->val);    // non-adjacent
        else if (first)    swap(first->val, middle->val);  // adjacent
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — one inorder pass |
| Space | O(H) recursion stack (O(1) extra with Morris traversal) |

> ⚠️ The adjacent-vs-non-adjacent distinction is the whole trick: if `last` stayed null, the swapped pair was adjacent, so swap `first`↔`middle`; otherwise swap `first`↔`last`.
