# Iterative Postorder (2 Stacks)  🟡

**Reference:** [LeetCode 145 — Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

---

## Problem
Postorder (**Left → Right → Root**) without recursion, using two stacks.

```
        1
       / \
      2   3        Postorder: 4 5 2 3 1
     / \
    4   5
```

---

## Intuition
Postorder is the reverse of a modified preorder (**Root → Right → Left**). Do that modified preorder pushing results onto a second stack; popping the second stack yields Left → Right → Root.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> postorderTraversal(TreeNode* root) {
    vector<int> out;
    if (!root) return out;
    stack<TreeNode*> s1, s2;
    s1.push(root);
    while (!s1.empty()) {
        TreeNode* node = s1.top(); s1.pop();
        s2.push(node);                     // collect in Root-Right-Left order
        if (node->left)  s1.push(node->left);
        if (node->right) s1.push(node->right);
    }
    while (!s2.empty()) {                  // reverse -> Left-Right-Root
        out.push_back(s2.top()->val);
        s2.pop();
    }
    return out;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) — second stack holds all nodes |

> Note the push order into `s1` here is **left then right** (opposite of preorder), because `s2` reverses it.
