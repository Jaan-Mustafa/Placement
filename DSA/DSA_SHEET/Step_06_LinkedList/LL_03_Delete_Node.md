# Delete a Node in Linked List  🟢

**Reference:** [GeeksforGeeks — Deletion in Linked List](https://www.geeksforgeeks.org/deletion-in-linked-list/) · [LeetCode 237](https://leetcode.com/problems/delete-node-in-a-linked-list/)

---

## Problem
Delete a node by position or value.

---

## Code (C++)
```cpp
// Delete head — O(1)
ListNode* deleteHead(ListNode* head) {
    if (!head) return nullptr;
    ListNode* next = head->next;
    delete head;
    return next;
}

// Delete by value (first occurrence) — O(N)
ListNode* deleteValue(ListNode* head, int val) {
    ListNode* dummy = new ListNode(-1);
    dummy->next = head;
    ListNode* cur = dummy;
    while (cur->next) {
        if (cur->next->val == val) {
            ListNode* toDel = cur->next;
            cur->next = cur->next->next;     // bypass the node
            delete toDel;
            break;
        }
        cur = cur->next;
    }
    return dummy->next;
}
```

### Delete a given node without head access (LeetCode 237)
You can't reach the previous node, so **copy the next node's value** into the current node and delete the next:
```cpp
void deleteNode(ListNode* node) {
    node->val = node->next->val;
    ListNode* tmp = node->next;
    node->next = node->next->next;
    delete tmp;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) (O(1) for head / given-node trick) |
| Space | O(1) |

> The **dummy node** removes the special-case for deleting the head.
