# Iterative Inorder Traversal  🟡

**Reference:** [LeetCode 94 — Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

---

## Problem
Inorder (**Left → Root → Right**) without recursion, using an explicit stack.

```
        1
       / \
      2   3        Inorder: 4 2 5 1 3
     / \
    4   5
```

---

## Intuition
Keep a pointer `cur`. Walk left as far as possible, pushing every node. When you can't go left, pop a node (that's the next inorder node), emit it, then move `cur` to its right child and repeat.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> out;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) {                  // go all the way left
            st.push(cur);
            cur = cur->left;
        }
        cur = st.top(); st.pop();      // leftmost unvisited node
        out.push_back(cur->val);       // Root
        cur = cur->right;              // then explore right
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

> The loop condition needs **both** `cur` and `!st.empty()` — dropping either ends traversal early.
