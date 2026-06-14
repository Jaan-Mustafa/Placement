# Reverse Nodes in K-Group  🔴

**Reference:** [LeetCode 25 — Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

---

## Problem
Reverse the nodes of the list `k` at a time. If the remaining nodes are fewer than `k`, leave them as is.

```
Input:  1 → 2 → 3 → 4 → 5,  k = 2    Output: 2 → 1 → 4 → 3 → 5
Input:  1 → 2 → 3 → 4 → 5,  k = 3    Output: 3 → 2 → 1 → 4 → 5
```

---

## Intuition
1. Check that at least `k` nodes remain (else stop — leave the tail untouched).
2. Reverse the next `k` nodes (standard iterative reversal bounded to k steps).
3. Recursively process the rest, attaching the head of the reversed remainder to the tail of this group.

---

## Code (C++)
```cpp
ListNode* reverseKGroup(ListNode* head, int k) {
    // 1. check k nodes exist
    ListNode* node = head;
    for (int i = 0; i < k; i++) {
        if (!node) return head;          // fewer than k → keep as is
        node = node->next;
    }
    // 2. reverse first k nodes
    ListNode* prev = nullptr;
    ListNode* cur = head;
    for (int i = 0; i < k; i++) {
        ListNode* next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }
    // 3. head is now the tail of this group; link to reversed remainder
    head->next = reverseKGroup(cur, k);
    return prev;                          // new head of this group
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each node reversed once |
| Space | O(N/k) recursion (can be made O(1) iteratively) |

> The pre-check loop is essential: it guarantees we only reverse **complete** groups of k.
