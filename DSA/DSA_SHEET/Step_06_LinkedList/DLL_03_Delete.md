# Delete a Node in a Doubly Linked List  🟢

**Reference:** [GeeksforGeeks — Delete a node in DLL](https://www.geeksforgeeks.org/delete-a-node-in-a-doubly-linked-list/)

---

## Problem
Delete a given node from a DLL in O(1) (since we have access to both neighbors via `prev`/`next`).

---

## Code (C++)
```cpp
Node* deleteNode(Node* head, Node* del) {
    if (!head || !del) return head;

    if (del->prev) del->prev->next = del->next;   // link prev → next
    else head = del->next;                        // deleting head

    if (del->next) del->next->prev = del->prev;   // link next → prev

    delete del;
    return head;
}
```

---

## Why DLL deletion is O(1)
In a singly list you must walk to find the predecessor (O(N)). In a DLL, `del->prev` gives it directly, so you can splice the node out immediately — this is the core advantage that LRU caches exploit.

## Complexity
| | |
|---|---|
| Time | O(1) given the node |
| Space | O(1) |

> Handle both edge cases: deleting the **head** (`del->prev == null`) and the **tail** (`del->next == null`).
