# Search an Element in a Linked List  🟢

**Reference:** [GeeksforGeeks — Search in a linked list](https://www.geeksforgeeks.org/search-an-element-in-a-linked-list-iterative-and-recursive/)

---

## Problem
Return whether a given value exists in the linked list.

```
List: 1 → 3 → 5 → 7,  target = 5    Output: true
List: 1 → 3 → 5 → 7,  target = 4    Output: false
```

---

## Code (C++)
```cpp
bool search(ListNode* head, int target) {
    while (head) {
        if (head->val == target) return true;
        head = head->next;
    }
    return false;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> No binary search possible — linked lists have no random access, so search is always linear.
