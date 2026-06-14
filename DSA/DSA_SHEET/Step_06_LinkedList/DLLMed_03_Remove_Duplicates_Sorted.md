# Remove Duplicates from a Sorted DLL  🟡

**Reference:** [GeeksforGeeks — Remove duplicates from a sorted DLL](https://www.geeksforgeeks.org/remove-duplicates-from-a-sorted-doubly-linked-list/)

---

## Problem
Given a sorted doubly linked list, remove duplicate nodes so each value appears once.

```
Input:  1 ⇄ 1 ⇄ 1 ⇄ 2 ⇄ 3 ⇄ 3 ⇄ 4
Output: 1 ⇄ 2 ⇄ 3 ⇄ 4
```

---

## Intuition
Since the list is sorted, duplicates are adjacent. For each node, skip forward over all nodes with the same value, relinking around them. Update the `prev` pointer of the surviving next node.

---

## Code (C++)
```cpp
Node* removeDuplicates(Node* head) {
    Node* cur = head;
    while (cur && cur->next) {
        if (cur->next->val == cur->val) {           // duplicate ahead
            Node* dup = cur->next;
            cur->next = dup->next;                  // skip duplicate
            if (dup->next) dup->next->prev = cur;    // fix backward link
            delete dup;
        } else {
            cur = cur->next;                        // move on
        }
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

> Don't advance `cur` after a deletion — there may be more duplicates of the same value right after it.
