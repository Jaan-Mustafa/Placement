# Reverse a Linked List (Iterative)  🟡

**Reference:** [LeetCode 206 — Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

---

## Problem
Reverse a singly linked list.

```
1 → 2 → 3 → 4 → null    →    4 → 3 → 2 → 1 → null
```

---

## Intuition
Walk the list with three pointers: `prev`, `cur`, `next`. For each node, save `next`, flip `cur->next` to point back at `prev`, then advance all pointers. When `cur` is null, `prev` is the new head.

---

## Code (C++)
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;
    while (cur) {
        ListNode* next = cur->next;   // save next
        cur->next = prev;             // reverse the link
        prev = cur;                   // advance prev
        cur = next;                   // advance cur
    }
    return prev;                      // new head
}
```

---

## Dry run
`1→2→3`: 
- prev=null,cur=1 → 1→null, prev=1,cur=2
- 2→1, prev=2,cur=3
- 3→2, prev=3,cur=null → head=**3→2→1**

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Memorize the four-line loop body — it's the most-asked linked list operation in interviews.
