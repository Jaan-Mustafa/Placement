# Find Pairs with Given Sum in a Sorted DLL  🟡

**Reference:** [GeeksforGeeks — Find pairs with given sum in DLL](https://www.geeksforgeeks.org/find-pairs-given-sum-doubly-linked-list/)

---

## Problem
Given a **sorted** doubly linked list, find all pairs of nodes whose values sum to a target.

```
Input:  1 ⇄ 2 ⇄ 4 ⇄ 5 ⇄ 6 ⇄ 8 ⇄ 9,  target = 7
Output: (1,6) (2,5)
```

---

## Intuition: two pointers from both ends
Because a DLL is sorted and bidirectional, we can use two pointers — `left` at the head, `right` at the tail (reached via `next`). Move them based on the sum, exactly like the two-pointer technique on a sorted array:
- sum == target → record pair, move both inward.
- sum < target → move `left` forward (`next`).
- sum > target → move `right` backward (`prev`).

---

## Code (C++)
```cpp
vector<pair<int,int>> pairsWithSum(Node* head, int target) {
    if (!head) return {};
    Node* right = head;
    while (right->next) right = right->next;   // go to the tail

    Node* left = head;
    vector<pair<int,int>> res;
    while (left != right && right->next != left) {   // not crossed
        int sum = left->val + right->val;
        if (sum == target) {
            res.push_back({left->val, right->val});
            left = left->next;
            right = right->prev;
        } else if (sum < target) {
            left = left->next;
        } else {
            right = right->prev;
        }
    }
    return res;
}
```

---

## Why DLL helps
The backward `prev` pointer lets `right` move left in O(1) — impossible in a singly list, which is why this two-pointer trick is a DLL specialty.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
