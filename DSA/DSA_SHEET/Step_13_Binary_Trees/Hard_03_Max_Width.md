# Maximum Width of Binary Tree  🟡

**Reference:** [LeetCode 662 — Maximum Width of Binary Tree](https://leetcode.com/problems/maximum-width-of-binary-tree/)

---

## Problem
The width of a level is the distance between the leftmost and rightmost non-null nodes, counting the null gaps between them. Return the maximum width over all levels.

```
        1
       / \
      3   2         Level widths: 1, 2, 4 -> max width = 4
     / \   \
    5   3   9
```

---

## Intuition
Assign each node a position index as if the tree were complete: left child = `2*i`, right child = `2*i + 1`. BFS level by level; width = `last - first + 1`. **Normalize** indices each level (subtract the level's first index) to prevent overflow.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
int widthOfBinaryTree(TreeNode* root) {
    if (!root) return 0;
    long long maxW = 0;
    queue<pair<TreeNode*, long long>> q;     // {node, index}
    q.push({root, 0});
    while (!q.empty()) {
        int sz = q.size();
        long long first = q.front().second;  // index of leftmost in level
        long long last  = first;
        for (int i = 0; i < sz; i++) {
            auto [node, idx] = q.front(); q.pop();
            idx -= first;                    // normalize to avoid overflow
            last = idx;
            if (node->left)  q.push({node->left,  2*idx});
            if (node->right) q.push({node->right, 2*idx + 1});
        }
        maxW = max(maxW, last + 1);
    }
    return (int)maxW;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> Without normalizing indices per level, a deep tree overflows even 64-bit ints. Subtract the level's `first` index every level.
