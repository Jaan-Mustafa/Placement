# Merge M Sorted Lists  🟡

**Reference:** [LeetCode 23 — Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

---

## Problem
Given `M` sorted linked lists, merge them into one sorted list.

```
Input:  [[1,4,5], [1,3,4], [2,6]]
Output: [1,1,2,3,4,4,5,6]
```

---

## Intuition
Use a **min-heap holding one node per list** (the current frontier). Pop the smallest node, append it to the result, then push that node's `next` so its list stays represented. The heap size never exceeds `M`, so each of the `N` total nodes costs `O(log M)`. Store node pointers in the heap with a custom comparator on `val`.

---

## Code (C++)
```cpp
struct ListNode { int val; ListNode* next; };

ListNode* mergeKLists(vector<ListNode*>& lists) {
    // min-heap of node pointers, ordered by node value
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);

    for (ListNode* head : lists)
        if (head) pq.push(head);            // seed with each list's head

    ListNode dummy(0), *tail = &dummy;
    while (!pq.empty()) {
        ListNode* node = pq.top(); pq.pop();
        tail->next = node; tail = node;     // attach smallest
        if (node->next) pq.push(node->next);// keep that list represented
    }
    tail->next = nullptr;
    return dummy.next;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log M), N = total nodes, M = number of lists |
| Space | O(M) for the heap |

> ⚠️ The comparator uses `a->val > b->val` to make a **min**-heap (greater = min). Skip null heads when seeding, and terminate the merged list with `tail->next = nullptr` to avoid a dangling tail.
