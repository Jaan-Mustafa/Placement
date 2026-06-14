# Iterative Preorder Traversal  🟡

**Reference:** [LeetCode 144 — Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

---

## Problem
Preorder (**Root → Left → Right**) without recursion, using an explicit stack.

```
        1
       / \
      2   3        Preorder: 1 2 4 5 3
     / \
    4   5
```

---

## Intuition
Use a stack. Pop a node, emit it, then push its **right child first and left child second** — so the left child sits on top and is processed next, preserving Root-Left-Right order.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> out;
    if (!root) return out;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        out.push_back(node->val);              // Root
        if (node->right) st.push(node->right); // push right first
        if (node->left)  st.push(node->left);  // left on top -> visited next
    }
    return out;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) stack |

> Push **right before left** — getting the order wrong silently produces Root-Right-Left.
