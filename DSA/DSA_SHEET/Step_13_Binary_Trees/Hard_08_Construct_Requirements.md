# Requirements to Construct a Unique Tree  🟡

**Reference:** [GeeksforGeeks — Construct a Binary Tree from given traversals](https://www.geeksforgeeks.org/if-you-are-given-two-traversal-sequences-can-you-construct-the-binary-tree/)

---

## Problem
Determine which pairs of traversal sequences uniquely reconstruct a binary tree.

```
preorder + inorder   -> unique  ✅
postorder + inorder  -> unique  ✅
preorder + postorder -> NOT unique (ambiguous) ❌
```

---

## Intuition
**Inorder** is the key: it tells you what lies in the left vs right subtree. Combined with preorder (which gives the root first) or postorder (root last), you can split and recurse uniquely.

Preorder + postorder fails because you cannot distinguish a left-only child from a right-only child — both produce the same pre/post sequences.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
// Demonstration: only pairs INCLUDING inorder are uniquely constructible.
bool canConstructUnique(const string& a, const string& b) {
    // true iff exactly one of the two traversals is "inorder"
    bool aIn = (a == "inorder");
    bool bIn = (b == "inorder");
    return aIn ^ bIn;        // XOR: exactly one must be inorder
}
// canConstructUnique("preorder",  "inorder")  -> true
// canConstructUnique("postorder", "inorder")  -> true
// canConstructUnique("preorder",  "postorder")-> false
```

---

## Complexity
| | |
|---|---|
| Time | O(1) for the rule; O(N) for actual construction |
| Space | O(1) |

> Pre + post can be unique **only** if the tree is guaranteed full (every node has 0 or 2 children). In general it is ambiguous.
