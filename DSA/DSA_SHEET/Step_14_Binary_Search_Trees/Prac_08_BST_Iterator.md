# BST Iterator  🟡

**Reference:** [LeetCode 173 — Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/)

---

## Problem
Implement an iterator over a BST that returns values in **ascending** order:
- `next()` — return the next smallest value.
- `hasNext()` — whether a next value exists.

```
[7,3,15,null,null,9,20]
next() 3  next() 7  hasNext() true  next() 9 ...
```

---

## Intuition
A full inorder traversal into an array works but uses O(N) memory upfront. Instead simulate the inorder traversal lazily with a **stack** holding the path of left descendants. Initially push the leftmost spine. Each `next()` pops the top (the current smallest), then pushes the left spine of its right child. The stack never holds more than the tree height, giving **amortized O(1)** per `next()` and O(H) space.

---

## Code (C++)
```cpp
class BSTIterator {
    stack<TreeNode*> st;
    void pushLeft(TreeNode* node) {       // push node and all left descendants
        while (node) { st.push(node); node = node->left; }
    }
public:
    BSTIterator(TreeNode* root) { pushLeft(root); }

    int next() {
        TreeNode* node = st.top(); st.pop();   // smallest unvisited
        pushLeft(node->right);                 // its right subtree's left spine
        return node->val;
    }

    bool hasNext() { return !st.empty(); }
};
```

---

## Complexity
| | |
|---|---|
| Time | `next()` / `hasNext()` amortized O(1) (each node pushed & popped once) |
| Space | O(H) — stack holds at most one root-to-leaf path |

> The key insight: across all `next()` calls each node is pushed and popped exactly once, so the total work is O(N), making per-call cost amortized O(1) despite a worst-case O(H) single call.
