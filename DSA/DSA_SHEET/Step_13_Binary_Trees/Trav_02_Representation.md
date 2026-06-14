# Binary Tree Representation  🟢

**Reference:** [GeeksforGeeks — Binary Tree Representation](https://www.geeksforgeeks.org/binary-tree-representations/)

---

## Problem
Represent a binary tree in memory and (optionally) as an array for complete trees.

```
        1
       / \
      2   3
```

---

## Intuition
Two common representations:
1. **Linked (pointer) representation** — each node holds `left`/`right` pointers. This is the general-purpose form used everywhere.
2. **Array representation** — for a node at index `i` (1-based / 0-based variants), children live at fixed positions. Efficient only for complete/near-complete trees (e.g. heaps).

Array index rule (0-based): node `i` → left child `2*i + 1`, right child `2*i + 2`, parent `(i-1)/2`.

---

## Code (C++)
```cpp
// (1) Linked representation — uses the TreeNode struct from Trav_01.
TreeNode* root = new TreeNode(1);
root->left  = new TreeNode(2);
root->right = new TreeNode(3);

// (2) Array representation of a complete tree.
// arr = [1, 2, 3, 4, 5] represents:
//          1
//         / \
//        2   3
//       / \
//      4   5
int leftChild (const vector<int>& a, int i) { return 2*i + 1; } // index
int rightChild(const vector<int>& a, int i) { return 2*i + 2; }
int parent    (const vector<int>& a, int i) { return (i - 1) / 2; }
```

---

## Complexity
| | |
|---|---|
| Time | O(1) for index math; O(N) to build |
| Space | Linked: O(N) + pointers. Array: O(2^h) worst (sparse trees waste space) |

> Array form wastes huge space for skewed trees — prefer pointers unless the tree is guaranteed complete.
