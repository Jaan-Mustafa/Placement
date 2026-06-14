# Find the Starting Point of a Loop  🟡

**Reference:** [LeetCode 142 — Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)

---

## Problem
If the list has a cycle, return the node where the cycle begins; otherwise return null.

---

## Intuition: Floyd's, phase two
After slow and fast meet inside the cycle, reset one pointer to the head. Move **both one step at a time**; they meet exactly at the cycle's start.

**Why:** if the distance from head to cycle-start is `L` and meeting point is `k` into the cycle, the math shows `L ≡ (cycle_length − k)`, so a pointer from head and one from the meeting point converge at the entry.

---

## Code (C++)
```cpp
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {                 // cycle detected
            ListNode* ptr = head;
            while (ptr != slow) {           // walk both 1 step
                ptr = ptr->next;
                slow = slow->next;
            }
            return ptr;                     // cycle start
        }
    }
    return nullptr;                         // no cycle
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Phase 1 detects the cycle (meeting point). Phase 2 — head pointer + meeting pointer at equal speed — pinpoints the entry node.
