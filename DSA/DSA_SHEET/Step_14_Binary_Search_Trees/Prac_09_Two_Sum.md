# Two Sum in BST  🟡

**Reference:** [LeetCode 653 — Two Sum IV - Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/)

---

## Problem
Given a BST and a target `k`, return whether two **distinct** nodes exist whose values sum to `k`.

```
Tree: [5,3,6,2,4,null,7], k = 9   →   true   (2 + 7 or 4 + 5)
Tree: same, k = 28                →   false
```

---

## Intuition
An inorder traversal gives sorted values, and sorted + two-pointer is the classic Two Sum. To avoid materializing the whole array, run **two iterators at once**: a normal inorder iterator (ascending, `next`) and a reverse inorder iterator (descending, `prev`). Treat them as the two pointers — if the sum is too small advance the ascending one, if too large advance the descending one, stopping when they meet.

---

## Code (C++)
```cpp
class BSTTwoSum {
    stack<TreeNode*> lo, hi;
    void pushLeft(TreeNode* n)  { while (n) { lo.push(n); n = n->left;  } }
    void pushRight(TreeNode* n) { while (n) { hi.push(n); n = n->right; } }
public:
    bool findTarget(TreeNode* root, int k) {
        pushLeft(root); pushRight(root);
        TreeNode* l = lo.top(); TreeNode* r = hi.top();
        while (l->val < r->val) {                 // distinct nodes only
            int sum = l->val + r->val;
            if (sum == k) return true;
            if (sum < k) {                        // need a bigger small-side
                lo.pop(); pushLeft(l->right); l = lo.top();
            } else {                              // need a smaller big-side
                hi.pop(); pushRight(r->left);  r = hi.top();
            }
        }
        return false;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each node advanced through at most once per pointer |
| Space | O(H) — two stacks, each at most one path deep |

> A simpler O(N) time / O(N) space option: inorder into a vector, then standard two-pointer. The dual-stack version above trades a little complexity for O(H) space.
