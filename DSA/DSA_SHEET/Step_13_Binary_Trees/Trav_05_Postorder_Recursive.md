# Postorder Traversal (Recursive)  🟢

**Reference:** [LeetCode 145 — Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

---

## Problem
Visit nodes in **Left → Right → Root** order.

```
        1
       / \
      2   3        Postorder: 4 5 2 3 1
     / \
    4   5
```

---

## Intuition
Recurse into both subtrees first, emit the node last. The node is processed only after all its descendants — ideal for "bottom-up" computations like deleting a tree or computing subtree aggregates.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void postorder(TreeNode* node, vector<int>& out) {
    if (!node) return;            // base case
    postorder(node->left,  out);  // Left
    postorder(node->right, out);  // Right
    out.push_back(node->val);     // Root
}

vector<int> postorderTraversal(TreeNode* root) {
    vector<int> out;
    postorder(root, out);
    return out;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> Postorder is the safe order to free/delete a tree — children are released before the parent.
