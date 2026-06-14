# Find the Length of a Linked List  🟢

**Reference:** [GeeksforGeeks — Find length of a linked list](https://www.geeksforgeeks.org/find-length-of-a-linked-list-iterative-and-recursive/)

---

## Problem
Count the number of nodes in a linked list.

```
1 → 2 → 3 → 4 → null    Output: 4
```

---

## Code (C++)
```cpp
// Iterative
int length(ListNode* head) {
    int count = 0;
    while (head) { count++; head = head->next; }
    return count;
}

// Recursive
int lengthRec(ListNode* head) {
    if (!head) return 0;
    return 1 + lengthRec(head->next);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) iterative / O(N) recursive (call stack) |
