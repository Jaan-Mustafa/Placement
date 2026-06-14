# Delete the Middle Node  🟡

**Reference:** [LeetCode 2095 — Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/)

---

## Problem
Delete the middle node (⌊N/2⌋-th, 0-indexed). If the list has one node, return null.

```
1 → 3 → 4 → 7 → 1 → 2 → 6    Output: 1 → 3 → 4 → 1 → 2 → 6  (7 deleted)
```

---

## Intuition
Use slow/fast pointers, but start `fast` two steps ahead (or skip one extra) so that when `fast` finishes, `slow` lands on the node **before** the middle — enabling its deletion.

---

## Code (C++)
```cpp
ListNode* deleteMiddle(ListNode* head) {
    if (!head || !head->next) return nullptr;     // single node → empty

    ListNode* slow = head;
    ListNode* fast = head->next->next;            // 2-step head start
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    ListNode* mid = slow->next;
    slow->next = slow->next->next;                // unlink middle
    delete mid;
    return head;
}
```

---

## Why offset `fast` by two?
Standard slow/fast lands `slow` **on** the middle. Here we need the node **before** the middle to relink, so we give `fast` a 2-node head start.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
