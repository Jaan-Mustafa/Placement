# Introduction to Trees  🟢

**Reference:** [GeeksforGeeks — Binary Tree Data Structure](https://www.geeksforgeeks.org/binary-tree-data-structure/)

---

## Problem
Understand what a binary tree is and its core terminology before solving any tree problem.

A **binary tree** is a hierarchical structure where each node has at most two children (`left` and `right`).

```
        1          <- root
       / \
      2   3        <- 2,3 are children of 1
     / \
    4   5          <- 4,5 are leaf nodes
```

Key terms:
- **Root** — topmost node (no parent).
- **Leaf** — node with no children.
- **Height** — number of edges on the longest root-to-leaf path.
- **Depth** — edges from root to the node.
- **Full tree** — every node has 0 or 2 children.
- **Complete tree** — all levels filled except possibly the last, filled left to right.

---

## Intuition
We model a node with a value plus two child pointers. This single struct is reused across **every** binary tree problem in this step, so define it once here.

---

## Code (C++)
```cpp
// Standard binary tree node — referenced by every later page in this step.
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// Building the sample tree above:
TreeNode* buildSample() {
    TreeNode* root = new TreeNode(1);
    root->left  = new TreeNode(2);
    root->right = new TreeNode(3);
    root->left->left  = new TreeNode(4);
    root->left->right = new TreeNode(5);
    return root;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(1) per node construction |
| Space | O(N) to store N nodes |

> This `TreeNode` definition is assumed in all following pages and is not repeated.
