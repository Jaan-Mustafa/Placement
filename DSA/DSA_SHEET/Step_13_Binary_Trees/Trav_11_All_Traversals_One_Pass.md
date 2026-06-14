# All Traversals in One Pass  🟡

**Reference:** [GeeksforGeeks — Preorder, Inorder, Postorder in One Traversal](https://www.geeksforgeeks.org/preorder-postorder-and-inorder-in-one-traversal/)

---

## Problem
Compute preorder, inorder, and postorder simultaneously in a single iterative pass.

```
        1
       / \
      2   3        Pre: 1 2 4 5 3 | In: 4 2 5 1 3 | Post: 4 5 2 3 1
     / \
    4   5
```

---

## Intuition
Push each node onto a stack with a **state counter** (1, 2, or 3):
- State 1 → add to **preorder**, advance to 2, go left.
- State 2 → add to **inorder**, advance to 3, go right.
- State 3 → add to **postorder**, pop the node.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void allTraversals(TreeNode* root,
                   vector<int>& pre, vector<int>& in, vector<int>& post) {
    if (!root) return;
    stack<pair<TreeNode*, int>> st;       // {node, state}
    st.push({root, 1});
    while (!st.empty()) {
        auto& [node, state] = st.top();
        if (state == 1) {                 // first touch
            pre.push_back(node->val);
            state++;
            if (node->left) st.push({node->left, 1});
        } else if (state == 2) {          // second touch
            in.push_back(node->val);
            state++;
            if (node->right) st.push({node->right, 1});
        } else {                          // third touch
            post.push_back(node->val);
            st.pop();
        }
    }
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each node touched 3 times, still O(N) |
| Space | O(N) for the three output lists + O(H) stack |

> Taking `auto& [node, state]` by **reference** is essential — incrementing `state` must mutate the stack entry in place.
