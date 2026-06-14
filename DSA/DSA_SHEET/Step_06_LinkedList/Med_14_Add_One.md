# Add 1 to a Number Represented as a Linked List  🟡

**Reference:** [GeeksforGeeks — Add 1 to a number represented as linked list](https://www.geeksforgeeks.org/add-1-number-represented-linked-list/)

---

## Problem
A number's digits are stored in a linked list, most significant digit first. Add 1 to it.

```
1 → 5 → 9    Output: 1 → 6 → 0
9 → 9 → 9    Output: 1 → 0 → 0 → 0
```

---

## Intuition: reverse, add carry, reverse back
Adding 1 starts from the **least significant** digit (the tail). Reverse the list, add 1 with carry propagation, then reverse back. If a final carry remains, prepend a new head node.

---

## Code (C++)
```cpp
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    while (head) { ListNode* nx = head->next; head->next = prev; prev = head; head = nx; }
    return prev;
}

ListNode* addOne(ListNode* head) {
    head = reverse(head);
    ListNode* cur = head;
    int carry = 1;                          // the +1
    ListNode* last = nullptr;
    while (cur) {
        int sum = cur->val + carry;
        cur->val = sum % 10;
        carry = sum / 10;
        last = cur;
        cur = cur->next;
        if (carry == 0) break;
    }
    head = reverse(head);
    if (carry) {                            // e.g. 999 → 1000
        ListNode* node = new ListNode(carry);
        node->next = head;
        head = node;
    }
    return head;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> A recursion-based approach can add 1 during the unwind (without reversing), returning the carry up the stack.
