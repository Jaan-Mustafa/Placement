# Floor in a BST  🟡

**Reference:** [GeeksforGeeks — Floor in BST](https://www.geeksforgeeks.org/floor-and-ceil-from-a-bst/)

---

## Problem
The **floor** of a value `key` is the largest value in the BST that is **less than or equal to** `key`. Return `-1` if none exists.

```
Tree: [8,4,12,2,6,10,14], key = 7   →   floor = 6
Tree: same, key = 6                 →   floor = 6
Tree: same, key = 1                 →   floor = -1
```

---

## Intuition
Mirror image of ceil. Walk down keeping a `floor` candidate. If `node->val == key`, done. If `node->val > key`, too big → go **left**. If `node->val < key`, this node is a **candidate** floor → record it, then go **right** to find a larger-yet-still-≤ value.

---

## Code (C++)
```cpp
int findFloor(TreeNode* root, int key) {
    int floor = -1;
    while (root) {
        if (root->val == key) return root->val;   // exact match is best
        if (root->val > key) {
            root = root->left;                     // need something smaller
        } else {
            floor = root->val;                     // candidate, try larger
            root = root->right;
        }
    }
    return floor;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(H) — single root-to-leaf descent |
| Space | O(1) iterative |

> ⚠️ Floor and ceil differ only in the comparison direction and which child you descend after recording a candidate — keep them straight under interview pressure.
