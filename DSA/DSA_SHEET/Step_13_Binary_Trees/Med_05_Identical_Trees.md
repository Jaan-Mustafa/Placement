# Check if Two Trees are Identical  🟡

**Reference:** [LeetCode 100 — Same Tree](https://leetcode.com/problems/same-tree/)

---

## Problem
Return true if two binary trees have the same structure and the same node values.

```
   1     1            1     1
  / \   / \    ==      / \   /
 2   3 2   3          2   3 2     -> NOT identical
```

---

## Intuition
Recurse both trees together. They match iff: both nodes are null (matching empty), or both are non-null with equal values **and** their left and right subtrees are identical.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p && !q) return true;             // both empty -> match
    if (!p || !q) return false;            // exactly one empty -> mismatch
    if (p->val != q->val) return false;    // values differ
    return isSameTree(p->left,  q->left)   // recurse both subtrees
        && isSameTree(p->right, q->right);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — N = nodes in the smaller tree before mismatch |
| Space | O(H) recursion stack |

> Check the `!p || !q` (structure mismatch) case *before* dereferencing `p->val` — otherwise you risk a null dereference.
