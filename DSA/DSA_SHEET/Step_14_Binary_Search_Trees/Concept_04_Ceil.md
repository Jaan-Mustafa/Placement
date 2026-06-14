# Ceil in a BST  🟡

**Reference:** [GeeksforGeeks — Ceil in BST](https://www.geeksforgeeks.org/floor-and-ceil-from-a-bst/)

---

## Problem
The **ceil** of a value `key` is the smallest value in the BST that is **greater than or equal to** `key`. Return `-1` if none exists.

```
Tree: [8,4,12,2,6,10,14], key = 7   →   ceil = 8
Tree: same, key = 6                 →   ceil = 6
Tree: same, key = 15                →   ceil = -1
```

---

## Intuition
Walk down the tree keeping a `ceil` candidate. If `node->val == key`, it's the answer. If `node->val < key`, no node here qualifies → go **right** for a larger value. If `node->val > key`, this node is a **candidate** ceil → record it, then go **left** to find a possibly smaller-yet-still-≥ value.

---

## Code (C++)
```cpp
int findCeil(TreeNode* root, int key) {
    int ceil = -1;
    while (root) {
        if (root->val == key) return root->val;   // exact match is best
        if (root->val < key) {
            root = root->right;                    // need something larger
        } else {
            ceil = root->val;                      // candidate, try smaller
            root = root->left;
        }
    }
    return ceil;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — single root-to-leaf descent |
| Space | O(1) iterative |

> The trick is that going **left** after recording a candidate only ever finds values that are still ≥ key (parent was > key), so the candidate can only improve.
