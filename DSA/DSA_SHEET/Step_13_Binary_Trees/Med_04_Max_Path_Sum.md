# Maximum Path Sum  🔴

**Reference:** [LeetCode 124 — Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

---

## Problem
Find the maximum sum of any non-empty path. A path is a sequence of connected nodes; it need not pass through the root. Node values may be negative.

```
       -10
       /  \
      9    20         Best path: 15 -> 20 -> 7 = 42
          /  \
        15    7
```

---

## Intuition
At each node, the best path that *bends* through it is `node->val + max(0, leftGain) + max(0, rightGain)` — update the global answer with this. But the value you *return* to the parent can only extend one branch: `node->val + max(0, max(leftGain, rightGain))`. Clamp negative gains to 0 (skip that branch).

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
int gain(TreeNode* node, int& best) {
    if (!node) return 0;
    int left  = max(0, gain(node->left,  best));  // drop negative branches
    int right = max(0, gain(node->right, best));
    best = max(best, node->val + left + right);    // path bending here
    return node->val + max(left, right);           // extend ONE side upward
}

int maxPathSum(TreeNode* root) {
    int best = INT_MIN;
    gain(root, best);
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> Initialize `best` to `INT_MIN`, not 0 — an all-negative tree's answer is its largest (least negative) node.
