# Reverse a Doubly Linked List  🟡

**Reference:** [GeeksforGeeks — Reverse a DLL](https://www.geeksforgeeks.org/reverse-a-doubly-linked-list/)

---

## Problem
Reverse a doubly linked list in place.

```
1 ⇄ 2 ⇄ 3 ⇄ 4    →    4 ⇄ 3 ⇄ 2 ⇄ 1
```

---

## Intuition
For each node, **swap its `next` and `prev` pointers**. After swapping all nodes, the original last node becomes the new head. Walk the list once, swapping pointers; the node whose (original) `prev` is null is the new tail/head boundary.

---

## Code (C++)
```cpp
Node* reverseDLL(Node* head) {
    Node* cur = head;
    Node* newHead = head;
    while (cur) {
        swap(cur->next, cur->prev);     // swap the two pointers
        newHead = cur;                  // track last processed node
        cur = cur->prev;                // prev is the OLD next now
    }
    return newHead;
}
```

---

## Why move with `cur->prev`?
After swapping, the node's `prev` field holds what used to be `next`. So to advance forward in the original order, we follow `cur->prev`.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
