# Middle of the Linked List  🟢

**Reference:** [LeetCode 876 — Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

---

## Problem
Return the middle node. If there are two middles (even length), return the second.

```
1 → 2 → 3 → 4 → 5         Output: 3
1 → 2 → 3 → 4 → 5 → 6     Output: 4
```

---

## Intuition: slow & fast pointers (tortoise–hare)
Move `slow` one step and `fast` two steps. When `fast` reaches the end, `slow` is at the middle. This finds the middle in **one pass** without first computing the length.

---

## Code (C++)
```cpp
ListNode* middleNode(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;          // 1 step
        fast = fast->next->next;    // 2 steps
    }
    return slow;
}
```

---

## Why it works
`fast` covers twice the distance of `slow`. When `fast` hits the end (distance N), `slow` has covered N/2 → the middle. For even length, this returns the **second** middle (start `fast = head`).

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Slow/fast pointers are the backbone of many LL problems: cycle detection, palindrome check, nth-from-end.
