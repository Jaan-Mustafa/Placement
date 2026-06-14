# Iterative Postorder (1 Stack)  🔴

**Reference:** [LeetCode 145 — Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

---

## Problem
Postorder (**Left → Right → Root**) using a single stack.

```
        1
       / \
      2   3        Postorder: 4 5 2 3 1
     / \
    4   5
```

---

## Intuition
Track a `last` pointer of the node visited most recently. Descend left pushing nodes. When the top has no right child (or its right was just visited), it's safe to emit it; otherwise move into the right subtree.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> postorderTraversal(TreeNode* root) {
    vector<int> out;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    TreeNode* last = nullptr;       // last node added to output
    while (cur || !st.empty()) {
        if (cur) {
            st.push(cur);
            cur = cur->left;        // dive left
        } else {
            TreeNode* peek = st.top();
            if (peek->right && last != peek->right) {
                cur = peek->right;  // right subtree not done yet
            } else {
                out.push_back(peek->val); // both children done -> emit
                last = peek;
                st.pop();
            }
        }
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

> The `last != peek->right` check prevents re-descending into an already-processed right subtree (infinite loop otherwise).
