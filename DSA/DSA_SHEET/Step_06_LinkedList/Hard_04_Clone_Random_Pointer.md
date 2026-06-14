# Clone a Linked List with Random + Next Pointers  🔴

**Reference:** [LeetCode 138 — Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)

---

## Problem
Each node has `next` and a `random` pointer (to any node or null). Make a **deep copy** of the list.

---

## Intuition: interleaving trick (O(1) extra space)
1. **Insert** each copy node right after its original: `A → A' → B → B' → ...`
2. **Assign randoms:** for each original `cur`, `cur->next->random = cur->random->next` (the copy's random is the next of the original's random).
3. **Separate** the two interleaved lists back into original and copy.

This avoids the O(N) hash map (`original → copy`) that the simpler solution uses.

---

## Code (C++)
```cpp
struct Node { int val; Node* next; Node* random; };

Node* copyRandomList(Node* head) {
    if (!head) return nullptr;

    // 1. interleave copies
    for (Node* cur = head; cur; cur = cur->next->next) {
        Node* copy = new Node(cur->val);
        copy->next = cur->next;
        cur->next = copy;
    }
    // 2. set random pointers on copies
    for (Node* cur = head; cur; cur = cur->next->next) {
        if (cur->random)
            cur->next->random = cur->random->next;
    }
    // 3. detach the copy list
    Node* dummy = new Node(0);
    Node* copyTail = dummy;
    for (Node* cur = head; cur; cur = cur->next) {
        copyTail->next = cur->next;        // copy node
        copyTail = copyTail->next;
        cur->next = cur->next->next;       // restore original
    }
    return dummy->next;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) extra (excluding the copy itself) |

> The hash-map version (map each original node to its clone, then wire up next/random) is easier to write and also O(N) time — but uses O(N) space. The interleaving trick is the optimal-space answer.
