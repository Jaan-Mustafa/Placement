# Height of Binary Tree  🟢

**Reference:** [LeetCode 104 — Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

---

## Problem
Return the height (max depth) of a binary tree — the number of nodes on the longest root-to-leaf path.

```
        1
       / \
      2   3        Height = 3
     / \
    4   5
```

---

## Intuition
Height of a node = 1 + max(height of left subtree, height of right subtree). An empty subtree has height 0. Pure postorder recursion.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;                   // empty subtree
    int lh = maxDepth(root->left);
    int rh = maxDepth(root->right);
    return 1 + max(lh, rh);                // this node + tallest child
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> "Height in nodes" returns 1 for a single node; "height in edges" returns 0. LeetCode 104 uses the node count.
