# Add Two Numbers (Linked Lists)  🟡

**Reference:** [LeetCode 2 — Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

---

## Problem
Two numbers are stored as linked lists with digits in **reverse** order (least significant first). Return their sum as a linked list.

```
A:  2 → 4 → 3   (342)
B:  5 → 6 → 4   (465)
Output: 7 → 0 → 8   (807)
```

---

## Intuition
Because digits are least-significant-first, we can add directly from the heads, propagating a carry — just like grade-school addition. A dummy head builds the result; continue while either list has digits or a carry remains.

---

## Code (C++)
```cpp
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy(-1), *tail = &dummy;
    int carry = 0;
    while (l1 || l2 || carry) {
        int sum = carry;
        if (l1) { sum += l1->val; l1 = l1->next; }
        if (l2) { sum += l2->val; l2 = l2->next; }
        carry = sum / 10;
        tail->next = new ListNode(sum % 10);
        tail = tail->next;
    }
    return dummy.next;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(max(N, M)) |
| Space | O(max(N, M)) for the result |

> The `|| carry` in the loop condition handles a final carry-out (e.g. 5 + 5 = 10 → an extra node). Reverse-order storage is what makes the addition straightforward.
