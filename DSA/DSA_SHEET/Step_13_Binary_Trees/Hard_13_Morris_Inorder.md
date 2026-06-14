# Morris Inorder Traversal  🔴

**Reference:** [GeeksforGeeks — Inorder Tree Traversal without Recursion and Stack (Morris)](https://www.geeksforgeeks.org/inorder-tree-traversal-without-recursion-and-without-stack/)

---

## Problem
Inorder traversal in **O(1) extra space** using threaded links.

```
        1
       / \
      2   3        Inorder: 4 2 5 1 3
     / \
    4   5
```

---

## Intuition
Same threading as Morris preorder, but emit the node **when removing** the thread (i.e., after the entire left subtree has been visited) — that is the correct inorder moment. If there is no left child, emit immediately and go right.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
vector<int> inorderMorris(TreeNode* root) {
    vector<int> out;
    TreeNode* cur = root;
    while (cur) {
        if (!cur->left) {                  // no left -> emit, go right
            out.push_back(cur->val);
            cur = cur->right;
        } else {
            TreeNode* pred = cur->left;     // inorder predecessor
            while (pred->right && pred->right != cur)
                pred = pred->right;
            if (!pred->right) {             // create thread, go left
                pred->right = cur;
                cur = cur->left;
            } else {                        // thread exists -> left done
                pred->right = nullptr;      // remove thread
                out.push_back(cur->val);    // INORDER: emit after left subtree
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
| Time | O(N) — each edge visited at most twice |
| Space | O(1) |

> ⚠️ The ONLY difference from Morris preorder is *where* you emit: inorder prints in the thread-removal branch (after the left subtree), preorder prints in the thread-creation branch. Restoring the thread is mandatory.
