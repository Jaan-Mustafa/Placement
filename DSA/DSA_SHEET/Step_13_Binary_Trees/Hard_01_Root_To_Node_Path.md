# Root to Node Path  🟡

**Reference:** [GeeksforGeeks — Print path from root to a given node](https://www.geeksforgeeks.org/print-path-from-root-to-a-given-node-in-a-binary-tree/)

---

## Problem
Given a target value, return the path from the root to that node.

```
        1
       / \
      2   3        Path to 5: [1, 2, 5]
     / \
    4   5
```

---

## Intuition
DFS carrying the current path. Push the node; if it's the target, succeed. Otherwise recurse into both children — if either finds the target, propagate success. If neither does, **backtrack** by popping the node.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
bool getPath(TreeNode* node, int target, vector<int>& path) {
    if (!node) return false;
    path.push_back(node->val);                 // choose
    if (node->val == target) return true;      // found
    if (getPath(node->left,  target, path) ||  // explore children
        getPath(node->right, target, path))
        return true;
    path.pop_back();                           // backtrack
    return false;
}

vector<int> rootToNode(TreeNode* root, int target) {
    vector<int> path;
    getPath(root, target, path);
    return path;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion + path |

> The `pop_back()` on failure is the backtracking step — omit it and unrelated branches leak into the path.
