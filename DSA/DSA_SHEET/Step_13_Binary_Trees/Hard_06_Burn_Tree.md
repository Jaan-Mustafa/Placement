# Minimum Time to Burn the Tree  🔴

**Reference:** [GeeksforGeeks — Minimum time to burn a Tree starting from a Leaf node](https://www.geeksforgeeks.org/minimum-time-to-burn-a-tree-starting-from-a-leaf-node/)

---

## Problem
Fire starts at a given node and spreads to adjacent nodes (parent and children) each unit of time. Return the time to burn the whole tree.

```
        1
       / \
      2   3        Start at 2 -> time to burn all = 3
     / \
    4   5
```

---

## Intuition
Identical structure to "nodes at distance K": build a parent map so fire can spread upward too, then BFS outward from the start node. The answer is the number of BFS levels needed minus one (the last level adds no further spread).

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
TreeNode* mapParents(TreeNode* root, int start,
                     unordered_map<TreeNode*, TreeNode*>& par) {
    TreeNode* startNode = nullptr;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* n = q.front(); q.pop();
        if (n->val == start) startNode = n;
        if (n->left)  { par[n->left]  = n; q.push(n->left); }
        if (n->right) { par[n->right] = n; q.push(n->right); }
    }
    return startNode;
}

int timeToBurnTree(TreeNode* root, int start) {
    unordered_map<TreeNode*, TreeNode*> par;
    TreeNode* src = mapParents(root, start, par);

    unordered_map<TreeNode*, bool> burnt;
    queue<TreeNode*> q; q.push(src);
    burnt[src] = true;
    int time = 0;
    while (!q.empty()) {
        int sz = q.size();
        bool spread = false;
        for (int i = 0; i < sz; i++) {
            TreeNode* n = q.front(); q.pop();
            for (TreeNode* nb : {n->left, n->right, par.count(n) ? par[n] : nullptr})
                if (nb && !burnt[nb]) { burnt[nb] = true; q.push(nb); spread = true; }
        }
        if (spread) time++;            // count a level only if fire advanced
    }
    return time;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> Increment `time` only when the fire actually spread this level; otherwise the final empty drain would over-count by one.
