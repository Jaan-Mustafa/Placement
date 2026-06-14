# Detect a Loop / Cycle in a Linked List  🟡

**Reference:** [LeetCode 141 — Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

---

## Problem
Determine whether the linked list contains a cycle.

```
3 → 2 → 0 → -4
        ↑________|     Output: true (cycle)
```

---

## Intuition: Floyd's cycle detection
Use slow (1 step) and fast (2 steps) pointers. If there's a cycle, `fast` will eventually lap `slow` and they'll meet. If `fast` reaches null, there's no cycle.

---

## Code (C++)
```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;     // pointers met → cycle
    }
    return false;                          // fast hit the end → no cycle
}
```

---

## Why they must meet
Inside a cycle, `fast` gains one position on `slow` each step. The gap shrinks by 1 every iteration, so they're guaranteed to collide — no infinite loop.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> A hash set of visited nodes also works (O(N) space). Floyd's is the O(1)-space standard.
