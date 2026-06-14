# Serialize and Deserialize Binary Tree  🔴

**Reference:** [LeetCode 297 — Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

---

## Problem
Encode a binary tree into a string and decode it back into the identical tree.

```
        1
       / \
      2   3        serialize -> "1,2,#,#,3,4,#,#,5,#,#,"
         / \
        4   5
```

---

## Intuition
Use BFS (level order). Serialize each node's value, writing `#` for nulls. Deserialize by reading tokens in the same order: dequeue a parent, attach the next two tokens as its children (skip `#`), enqueue real children.

(`TreeNode` defined in Trav_01.)

---

## Code (C++)
```cpp
string serialize(TreeNode* root) {
    if (!root) return "";
    string s;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* n = q.front(); q.pop();
        if (!n) { s += "#,"; continue; }
        s += to_string(n->val) + ",";
        q.push(n->left);
        q.push(n->right);
    }
    return s;
}

TreeNode* deserialize(string data) {
    if (data.empty()) return nullptr;
    stringstream ss(data);
    string tok;
    getline(ss, tok, ',');
    TreeNode* root = new TreeNode(stoi(tok));
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* parent = q.front(); q.pop();
        if (getline(ss, tok, ',') && tok != "#") {     // left child
            parent->left = new TreeNode(stoi(tok));
            q.push(parent->left);
        }
        if (getline(ss, tok, ',') && tok != "#") {     // right child
            parent->right = new TreeNode(stoi(tok));
            q.push(parent->right);
        }
    }
    return root;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) for both directions |
| Space | O(N) string + queue |

> Serialize and deserialize must agree on the **exact** order and null marker; mixing a preorder serializer with a level-order parser corrupts the tree.
