# Flattening a Linked List  🔴

**Reference:** [GeeksforGeeks — Flattening a Linked List](https://www.geeksforgeeks.org/flattening-a-linked-list/)

---

## Problem
Each node has a `next` pointer (to the next sub-list head) and a `bottom` pointer (a sorted vertical list). Flatten everything into a single **sorted** list using `bottom` pointers only.

```
5 → 10 → 19 → 28
|    |    |    |
7    20   22   35
|         |    |
8         50   40
|              |
30             45

Output: 5→7→8→10→19→20→22→28→30→35→40→45→50
```

---

## Intuition
Each vertical (`bottom`) list is already sorted. Merge them pairwise like merging sorted lists, recursing from the rightmost sub-list back to the left, repeatedly merging the current list with the flattened remainder.

---

## Code (C++)
```cpp
struct Node { int val; Node* next; Node* bottom; };

Node* merge(Node* a, Node* b) {
    if (!a) return b;
    if (!b) return a;
    Node* res;
    if (a->val <= b->val) { res = a; res->bottom = merge(a->bottom, b); }
    else                  { res = b; res->bottom = merge(a, b->bottom); }
    res->next = nullptr;
    return res;
}

Node* flatten(Node* root) {
    if (!root || !root->next) return root;
    root->next = flatten(root->next);          // flatten the rest first
    root = merge(root, root->next);            // merge with current
    return root;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · M) — total nodes across all merges |
| Space | O(N) recursion (merge depth) |

> Merging happens on the `bottom` pointer; the final list uses only `bottom` (all `next` set to null).
