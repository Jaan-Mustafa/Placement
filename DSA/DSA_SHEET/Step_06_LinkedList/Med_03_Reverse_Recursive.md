# Reverse a Linked List (Recursive)  🟡

**Reference:** [LeetCode 206 — Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

---

## Problem
Reverse a singly linked list using recursion.

```
1 → 2 → 3 → 4 → null    →    4 → 3 → 2 → 1 → null
```

---

## Intuition
Recurse to the end first; the last node becomes the new head. While unwinding, make the **next** node point back at the current node (`head->next->next = head`) and sever the forward link (`head->next = nullptr`).

---

## Code (C++)
```cpp
ListNode* reverseList(ListNode* head) {
    if (!head || !head->next) return head;     // base: empty or single node

    ListNode* newHead = reverseList(head->next);   // reverse the rest
    head->next->next = head;                        // make next point back
    head->next = nullptr;                           // sever old forward link
    return newHead;                                 // unchanged through unwinding
}
```

---

## How the relinking works
For `1→2→3`, after `reverseList(2→3)` returns head `3` (with `3→2→null`), we set `1`'s next-node (`2`) to point back: `2->next = 1`, and `1->next = null`. Result `3→2→1`.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |

> The iterative version is O(1) space — prefer it in practice, but interviewers often ask for both.
