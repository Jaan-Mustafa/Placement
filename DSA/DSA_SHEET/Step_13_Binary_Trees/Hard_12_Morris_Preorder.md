# Morris Preorder Traversal  🔴

**Reference:** [GeeksforGeeks — Morris traversal for Preorder](https://www.geeksforgeeks.org/morris-traversal-for-preorder/)

---

## Problem
Preorder traversal in **O(1) extra space** — no recursion, no stack — using threaded binary tree links.

```
        1
       / \
      2   3        Preorder: 1 2 4 5 3
     / \
    4   5
```

---

## Intuition
For each node with a left child, find its **inorder predecessor** (rightmost node of the left subtree). Temporarily thread that predecessor's right pointer back to the current node so we can return. For preorder, emit the node **when creating** the thread (before going left). Remove the thread when we come back.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> preorderMorris(TreeNode* root) {
    vector<int> out;
    TreeNode* cur = root;
    while (cur) {
        if (!cur->left) {                  // no left -> emit, go right
            out.push_back(cur->val);
            cur = cur->right;
        } else {
            TreeNode* pred = cur->left;     // find inorder predecessor
            while (pred->right && pred->right != cur)
                pred = pred->right;
            if (!pred->right) {             // create thread
                out.push_back(cur->val);    // PREORDER: emit before going left
                pred->right = cur;
                cur = cur->left;
            } else {                        // thread exists -> remove it
                pred->right = nullptr;
                cur = cur->right;
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
| Time | O(N) — each edge traversed at most twice |
| Space | O(1) — no stack/recursion |

> ⚠️ Morris **mutates** the tree's right pointers while running. You MUST restore each thread (the `pred->right = nullptr` branch), or the tree is left corrupted and traversal can loop.
