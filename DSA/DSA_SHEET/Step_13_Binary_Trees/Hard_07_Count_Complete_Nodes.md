# Count Total Nodes in Complete Tree  🟡

**Reference:** [LeetCode 222 — Count Complete Tree Nodes](https://leetcode.com/problems/count-complete-tree-nodes/)

---

## Problem
Count the nodes of a **complete** binary tree faster than O(N).

```
        1
       / \
      2   3        Count = 6
     / \  /
    4  5 6
```

---

## Intuition
Measure the left height (always going left) and right height (always going right). If they're equal, the subtree is a perfect tree with `2^h - 1` nodes — return instantly. Otherwise recurse on both children and add 1.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
int leftHeight(TreeNode* n)  { int h = 0; while (n) { h++; n = n->left;  } return h; }
int rightHeight(TreeNode* n) { int h = 0; while (n) { h++; n = n->right; } return h; }

int countNodes(TreeNode* root) {
    if (!root) return 0;
    int lh = leftHeight(root);
    int rh = rightHeight(root);
    if (lh == rh)                          // perfect subtree
        return (1 << lh) - 1;              // 2^lh - 1 nodes
    return 1 + countNodes(root->left) + countNodes(root->right);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log²N) — O(log N) heights at O(log N) recursion levels |
| Space | O(log N) recursion stack |

> The `lh == rh` shortcut is what beats the naive O(N) count — at most one node per level breaks perfection, so recursion only branches down one spine.
