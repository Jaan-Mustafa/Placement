# Intersection of Two Linked Lists  🟡

**Reference:** [LeetCode 160 — Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/)

---

## Problem
Return the node where two singly linked lists intersect, or null if they don't. (Intersection means the same node object, not equal values.)

```
A:      4 → 1 ↘
                 8 → 4 → 5
B:  5 → 6 → 1 ↗            Output: node with value 8
```

---

## Intuition: two-pointer length equalizer
Two pointers `a` and `b` traverse their lists. When one reaches the end, redirect it to the **other** list's head. After at most two passes, both pointers have travelled `lenA + lenB` and align at the intersection (or both hit null together).

---

## Code (C++)
```cpp
ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    ListNode* a = headA;
    ListNode* b = headB;
    while (a != b) {
        a = a ? a->next : headB;      // switch to B at end of A
        b = b ? b->next : headA;      // switch to A at end of B
    }
    return a;                         // intersection node or null
}
```

---

## Why they align
After switching, both pointers walk `lenA + lenB` total. If lists intersect, the tail lengths after the junction are equal, so the pointers reach the junction simultaneously. If not, both become null at the same time → loop ends.

## Complexity
| | |
|---|---|
| Time | O(N + M) |
| Space | O(1) |

> A hash set of one list's nodes also works (O(N) space). The two-pointer trick is the elegant O(1)-space solution.
