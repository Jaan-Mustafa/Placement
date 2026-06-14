# Diameter of Binary Tree  🟡

**Reference:** [LeetCode 543 — Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)

---

## Problem
The diameter is the length (in **edges**) of the longest path between any two nodes; the path may or may not pass through the root.

```
        1
       / \
      2   3        Longest path: 4-2-1-3 -> diameter = 3 edges
     / \
    4   5
```

---

## Intuition
For each node, the longest path *through* it equals `leftHeight + rightHeight`. Compute heights bottom-up and keep a global max of `lh + rh` seen at every node.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
int height(TreeNode* node, int& diameter) {
    if (!node) return 0;
    int lh = height(node->left,  diameter);
    int rh = height(node->right, diameter);
    diameter = max(diameter, lh + rh);     // path through this node (edges)
    return 1 + max(lh, rh);
}

int diameterOfBinaryTree(TreeNode* root) {
    int diameter = 0;
    height(root, diameter);
    return diameter;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> The diameter is updated at *every* node, not just the root — the longest path often lives in a subtree.
