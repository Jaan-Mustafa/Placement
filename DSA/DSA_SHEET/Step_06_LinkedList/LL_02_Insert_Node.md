# Insert a Node in Linked List  🟢

**Reference:** [GeeksforGeeks — Insertion in Linked List](https://www.geeksforgeeks.org/insertion-in-linked-list/)

---

## Problem
Insert a node at the head, at the tail, or at a given position.

---

## Code (C++)
```cpp
// Insert at head — O(1)
ListNode* insertHead(ListNode* head, int val) {
    ListNode* node = new ListNode(val);
    node->next = head;
    return node;                 // new head
}

// Insert at tail — O(N)
ListNode* insertTail(ListNode* head, int val) {
    ListNode* node = new ListNode(val);
    if (!head) return node;
    ListNode* cur = head;
    while (cur->next) cur = cur->next;
    cur->next = node;
    return head;
}

// Insert at position k (1-indexed) — O(k)
ListNode* insertAtPos(ListNode* head, int val, int k) {
    if (k == 1) return insertHead(head, val);
    ListNode* cur = head;
    for (int i = 1; i < k - 1 && cur; i++) cur = cur->next;  // node before k
    ListNode* node = new ListNode(val);
    node->next = cur->next;
    cur->next = node;
    return head;
}
```

---

## Key idea
Always set the **new node's `next` first**, then relink the previous node — otherwise you lose the rest of the list.

## Complexity
| Operation | Time |
|---|---|
| At head | O(1) |
| At tail | O(N) |
| At position k | O(k) |
