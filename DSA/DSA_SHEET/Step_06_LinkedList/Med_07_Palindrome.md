# Check if Linked List is a Palindrome  🟡

**Reference:** [LeetCode 234 — Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

---

## Problem
Return true if the linked list reads the same forwards and backwards.

```
1 → 2 → 2 → 1    Output: true
1 → 2 → 3        Output: false
```

---

## Intuition (O(1) space)
1. Find the middle with slow/fast pointers.
2. Reverse the second half.
3. Compare the first half with the reversed second half node by node.
4. (Optionally restore the list by reversing again.)

---

## Code (C++)
```cpp
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    while (head) { ListNode* nx = head->next; head->next = prev; prev = head; head = nx; }
    return prev;
}

bool isPalindrome(ListNode* head) {
    if (!head || !head->next) return true;
    // 1. find middle
    ListNode* slow = head; ListNode* fast = head;
    while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
    // 2. reverse second half
    ListNode* second = reverse(slow->next);
    // 3. compare
    ListNode* p1 = head; ListNode* p2 = second;
    bool ok = true;
    while (p2) {
        if (p1->val != p2->val) { ok = false; break; }
        p1 = p1->next; p2 = p2->next;
    }
    slow->next = reverse(second);     // 4. restore (optional)
    return ok;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Simpler O(N) space alternative: copy values to a vector and check with two pointers. The reversal method is the optimal-space answer.
