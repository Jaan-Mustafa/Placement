# Sort a Linked List  🟡

**Reference:** [LeetCode 148 — Sort List](https://leetcode.com/problems/sort-list/)

---

## Problem
Sort a linked list in O(N log N) time and O(1) extra space (ignoring recursion stack).

```
4 → 2 → 1 → 3    Output: 1 → 2 → 3 → 4
```

---

## Intuition: merge sort on a linked list
Merge sort suits linked lists (no random access needed):
1. Split the list into two halves (find middle with slow/fast).
2. Recursively sort each half.
3. Merge the two sorted halves.

---

## Code (C++)
```cpp
ListNode* merge(ListNode* a, ListNode* b) {
    ListNode dummy(-1), *tail = &dummy;
    while (a && b) {
        if (a->val <= b->val) { tail->next = a; a = a->next; }
        else                  { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;
    return dummy.next;
}

ListNode* sortList(ListNode* head) {
    if (!head || !head->next) return head;
    // split: find middle
    ListNode* slow = head; ListNode* fast = head->next;
    while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
    ListNode* mid = slow->next;
    slow->next = nullptr;                 // cut into two halves

    ListNode* left = sortList(head);
    ListNode* right = sortList(mid);
    return merge(left, right);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(log N) recursion stack |

> Merge sort is preferred over quicksort for lists — splitting and merging only need sequential access, which lists provide naturally.
