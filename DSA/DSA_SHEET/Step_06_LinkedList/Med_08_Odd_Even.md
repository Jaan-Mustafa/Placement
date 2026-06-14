# Segregate Odd and Even Position Nodes  🟡

**Reference:** [LeetCode 328 — Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/)

---

## Problem
Group all nodes at **odd positions** together followed by nodes at **even positions** (by index, 1-based), preserving relative order. O(1) space.

```
1 → 2 → 3 → 4 → 5    Output: 1 → 3 → 5 → 2 → 4
```

---

## Intuition
Maintain two chains — `odd` and `even` — built by hopping two nodes at a time. At the end, attach the head of the even chain to the tail of the odd chain.

---

## Code (C++)
```cpp
ListNode* oddEvenList(ListNode* head) {
    if (!head || !head->next) return head;
    ListNode* odd = head;
    ListNode* even = head->next;
    ListNode* evenHead = even;           // remember start of even chain
    while (even && even->next) {
        odd->next = even->next;          // link next odd
        odd = odd->next;
        even->next = odd->next;          // link next even
        even = even->next;
    }
    odd->next = evenHead;                // attach evens after odds
    return head;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> This is "by position", not "by value". The two pointers weave through alternating nodes, then the chains are stitched together.
