# All Nodes at Distance K  🔴

**Reference:** [LeetCode 863 — All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

---

## Problem
Given a target node and integer `k`, return all node values exactly distance `k` from the target (distance counts edges, in any direction).

```
        3
       / \
      5   1        target = 5, k = 2  ->  [7, 4, 1]
     / \ / \
    6  2 0  8
      / \
     7   4
```

---

## Intuition
The tree only has downward pointers, but distance can go upward too. Build a **parent map** so we can walk up as well as down, then BFS outward from the target like an undirected graph, stopping at depth `k`.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void markParents(TreeNode* root, unordered_map<TreeNode*, TreeNode*>& par) {
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* n = q.front(); q.pop();
        if (n->left)  { par[n->left]  = n; q.push(n->left); }
        if (n->right) { par[n->right] = n; q.push(n->right); }
    }
}

vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
    unordered_map<TreeNode*, TreeNode*> par;
    markParents(root, par);

    unordered_map<TreeNode*, bool> visited;
    queue<TreeNode*> q; q.push(target);
    visited[target] = true;
    int dist = 0;
    while (!q.empty()) {
        if (dist == k) break;              // reached distance k
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode* n = q.front(); q.pop();
            for (TreeNode* nb : {n->left, n->right, par.count(n) ? par[n] : nullptr})
                if (nb && !visited[nb]) { visited[nb] = true; q.push(nb); }
        }
        dist++;
    }
    vector<int> res;
    while (!q.empty()) { res.push_back(q.front()->val); q.pop(); }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) — parent map + visited + queue |

> The `visited` set is mandatory in the multi-direction BFS — without it you bounce between a node and its parent forever.
