# Remove Nth Node from End  🟡

**Reference:** [LeetCode 19 — Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

---

## Problem
Remove the n-th node from the end of the list in **one pass**.

```
1 → 2 → 3 → 4 → 5,  n = 2    Output: 1 → 2 → 3 → 5
```

---

## Intuition: gap of n between two pointers
Advance `fast` by `n` nodes first. Then move `fast` and `slow` together until `fast` reaches the last node — now `slow` sits just before the target. A **dummy** node handles removing the head cleanly.

---

## Code (C++)
```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode* dummy = new ListNode(-1);
    dummy->next = head;
    ListNode* fast = dummy;
    ListNode* slow = dummy;

    for (int i = 0; i < n; i++) fast = fast->next;   // create the gap of n

    while (fast->next) {                             // move both to the end
        fast = fast->next;
        slow = slow->next;
    }
    ListNode* toDel = slow->next;                    // slow is before target
    slow->next = slow->next->next;
    delete toDel;
    return dummy->next;
}
```

---

## Why the dummy node?
If `n` equals the list length, the head must be removed. Starting both pointers at `dummy` makes `slow` land before the head in that case, avoiding a special case.

## Complexity
| | |
|---|---|
| Time | O(N) single pass |
| Space | O(1) |
