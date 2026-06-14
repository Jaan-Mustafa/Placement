# Children Sum Property  🔴

**Reference:** [GeeksforGeeks — Transform to Sum Tree (Children Sum Property)](https://www.geeksforgeeks.org/convert-an-arbitrary-binary-tree-to-a-tree-that-holds-children-sum-property/)

---

## Problem
Modify the tree so that every node's value equals the sum of its children's values. You may only **increment** node values (never decrement). Leaf values may rise.

```
        50                 50
       /  \      ->       /  \
      7    2            28    22   (each parent = sum of its children)
```

---

## Intuition
Two passes. **Down:** if a node's value exceeds its children's sum, push the excess down to the children (increase a child). If the children's sum exceeds the node, set the node to that sum. **Up (postorder):** after fixing children, set each node to the sum of its (now-final) children. This two-way flow guarantees the property without ever decreasing.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
void changeTree(TreeNode* root) {
    if (!root) return;
    if (!root->left && !root->right) return;          // leaf: nothing to do

    int childSum = 0;
    if (root->left)  childSum += root->left->val;
    if (root->right) childSum += root->right->val;

    if (childSum >= root->val) {
        root->val = childSum;                         // children dominate
    } else {
        // node dominates: push the larger value down to a child
        if (root->left)  root->left->val  = root->val;
        else if (root->right) root->right->val = root->val;
    }

    changeTree(root->left);                            // recurse down
    changeTree(root->right);

    int total = 0;                                     // postorder fix-up
    if (root->left)  total += root->left->val;
    if (root->right) total += root->right->val;
    if (root->left || root->right) root->val = total;  // set to children sum
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(H) recursion stack |

> The downward pass can over-credit a child; the postorder upward pass corrects every node to its true children sum. Both passes are required.
