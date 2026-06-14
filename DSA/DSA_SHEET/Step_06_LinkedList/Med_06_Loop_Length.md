# Length of the Loop in a Linked List  🟡

**Reference:** [GeeksforGeeks — Length of loop in linked list](https://www.geeksforgeeks.org/find-length-of-loop-in-linked-list/)

---

## Problem
If the list has a cycle, return the number of nodes in it; otherwise 0.

---

## Intuition
Use Floyd's to find a meeting point inside the loop. From there, keep one pointer fixed and walk another around the loop counting steps until it returns to the meeting point — that count is the loop length.

---

## Code (C++)
```cpp
int countNodesInLoop(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {                     // meeting point in loop
            int count = 1;
            ListNode* cur = slow->next;
            while (cur != slow) {               // walk around the loop
                cur = cur->next;
                count++;
            }
            return count;
        }
    }
    return 0;                                   // no loop
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
