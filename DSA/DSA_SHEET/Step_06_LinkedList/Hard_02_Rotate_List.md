# Rotate a Linked List  🔴

**Reference:** [LeetCode 61 — Rotate List](https://leetcode.com/problems/rotate-list/)

---

## Problem
Rotate the list to the right by `k` places.

```
Input:  1 → 2 → 3 → 4 → 5,  k = 2    Output: 4 → 5 → 1 → 2 → 3
```

---

## Intuition
1. Find the length `n` and the tail.
2. `k %= n` (rotations beyond `n` repeat).
3. Make the list **circular** by joining tail → head.
4. The new tail is at position `n - k`; the new head is the node after it. Break the circle there.

---

## Code (C++)
```cpp
ListNode* rotateRight(ListNode* head, int k) {
    if (!head || !head->next || k == 0) return head;

    // 1. length + tail
    int n = 1;
    ListNode* tail = head;
    while (tail->next) { tail = tail->next; n++; }

    // 2. effective rotations
    k %= n;
    if (k == 0) return head;

    // 3. make circular
    tail->next = head;

    // 4. new tail = (n - k)th node
    ListNode* newTail = head;
    for (int i = 1; i < n - k; i++) newTail = newTail->next;
    ListNode* newHead = newTail->next;
    newTail->next = nullptr;            // break the circle
    return newHead;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> The `k %= n` step avoids redundant full rotations — rotating by `n` returns the original list.
