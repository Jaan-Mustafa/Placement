# Sort a Linked List of 0s, 1s, and 2s  🟡

**Reference:** [GeeksforGeeks — Sort a LL of 0s 1s 2s](https://www.geeksforgeeks.org/sort-a-linked-list-of-0s-1s-or-2s/)

---

## Problem
Sort a linked list containing only values 0, 1, 2.

```
1 → 2 → 0 → 1 → 2 → 0    Output: 0 → 0 → 1 → 1 → 2 → 2
```

---

## Intuition: three separate lists
Create three dummy heads — for 0s, 1s, 2s. Traverse once, appending each node to the matching list. Finally, concatenate `zero → one → two`. This is a one-pass, O(1)-extra-space link rearrangement (no value swapping).

---

## Code (C++)
```cpp
ListNode* sortList012(ListNode* head) {
    ListNode zeroD(-1), oneD(-1), twoD(-1);
    ListNode *zero = &zeroD, *one = &oneD, *two = &twoD;

    for (ListNode* cur = head; cur; cur = cur->next) {
        if (cur->val == 0)      { zero->next = cur; zero = zero->next; }
        else if (cur->val == 1) { one->next = cur;  one = one->next; }
        else                    { two->next = cur;  two = two->next; }
    }
    // stitch: 0s → 1s → 2s
    zero->next = oneD.next ? oneD.next : twoD.next;
    one->next = twoD.next;
    two->next = nullptr;                  // terminate
    return zeroD.next;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) single pass |
| Space | O(1) |

> A counting approach (count 0s/1s/2s, then overwrite values) also works in O(N) but modifies data instead of relinking. The three-list method preserves node identity.
