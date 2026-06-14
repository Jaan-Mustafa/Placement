# Delete All Occurrences of a Key in a DLL  🟡

**Reference:** [GeeksforGeeks — Delete all occurrences of a key in DLL](https://www.geeksforgeeks.org/delete-occurrences-given-key-doubly-linked-list/)

---

## Problem
Given a doubly linked list and a key, delete **every** node whose value equals the key.

```
Input:  10 ⇄ 4 ⇄ 10 ⇄ 4 ⇄ 5,  key = 10
Output: 4 ⇄ 4 ⇄ 5
```

---

## Intuition
Traverse the DLL. For each node matching the key, splice it out by linking its `prev` and `next` to each other — O(1) per deletion thanks to the `prev` pointer. Carefully update the head when the matched node is the head.

---

## Code (C++)
```cpp
Node* deleteAllOccurrences(Node* head, int key) {
    Node* cur = head;
    while (cur) {
        Node* next = cur->next;             // save before deleting
        if (cur->val == key) {
            if (cur->prev) cur->prev->next = cur->next;
            else head = cur->next;          // deleting the head
            if (cur->next) cur->next->prev = cur->prev;
            delete cur;
        }
        cur = next;
    }
    return head;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Save `cur->next` **before** deleting — otherwise you lose the rest of the list after `delete cur`.
