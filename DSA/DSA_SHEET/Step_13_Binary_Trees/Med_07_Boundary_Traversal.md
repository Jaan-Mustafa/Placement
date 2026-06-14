# Boundary Traversal  🔴

**Reference:** [GeeksforGeeks — Boundary Traversal of Binary Tree](https://www.geeksforgeeks.org/boundary-traversal-of-binary-tree/)

---

## Problem
Print the boundary anticlockwise: root, then the left boundary (top→bottom, excluding leaves), then all leaves (left→right), then the right boundary (bottom→top, excluding leaves).

```
            1
          /   \
         2     3        Boundary: 1 2 4 5 6 7 3
        / \   / \
       4   5 6   7
```

---

## Intuition
Three independent collections, carefully avoiding double-counting leaves:
1. **Left boundary** (top-down) excluding leaf nodes.
2. **All leaves** left to right (inorder-ish DFS).
3. **Right boundary** (bottom-up) excluding leaf nodes.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
bool isLeaf(TreeNode* n) { return !n->left && !n->right; }

void addLeftBoundary(TreeNode* root, vector<int>& res) {
    TreeNode* cur = root->left;
    while (cur) {
        if (!isLeaf(cur)) res.push_back(cur->val);
        cur = cur->left ? cur->left : cur->right;   // prefer left
    }
}
void addLeaves(TreeNode* node, vector<int>& res) {
    if (!node) return;
    if (isLeaf(node)) { res.push_back(node->val); return; }
    addLeaves(node->left,  res);
    addLeaves(node->right, res);
}
void addRightBoundary(TreeNode* root, vector<int>& res) {
    TreeNode* cur = root->right;
    vector<int> tmp;
    while (cur) {
        if (!isLeaf(cur)) tmp.push_back(cur->val);
        cur = cur->right ? cur->right : cur->left;  // prefer right
    }
    for (int i = tmp.size() - 1; i >= 0; i--)        // reverse -> bottom-up
        res.push_back(tmp[i]);
}

vector<int> boundary(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    if (!isLeaf(root)) res.push_back(root->val);     // root (skip if it's a leaf)
    addLeftBoundary(root, res);
    addLeaves(root, res);
    addRightBoundary(root, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) — output + recursion |

> Exclude leaves from the left/right boundary walks, or they get printed twice. Also guard the root-is-leaf edge case.
