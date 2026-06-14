# Introduction to Linked List  🟢

**Reference:** [GeeksforGeeks — Linked List](https://www.geeksforgeeks.org/data-structures/linked-list/)

---

## What is a Linked List?
A linear data structure where elements (**nodes**) are stored in non-contiguous memory; each node holds data plus a pointer to the next node. Unlike arrays, there's no random access, but insertion/deletion at a known position is O(1).

```
head → [1|•] → [2|•] → [3|null]
```

---

## Node definition (used across all LL problems)
```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
```

---

## Build a linked list from an array
```cpp
ListNode* buildList(vector<int>& arr) {
    ListNode* dummy = new ListNode(-1);
    ListNode* tail = dummy;
    for (int x : arr) {
        tail->next = new ListNode(x);
        tail = tail->next;
    }
    return dummy->next;     // first real node
}
```

---

## Traverse and print
```cpp
void printList(ListNode* head) {
    while (head) {
        cout << head->val << " ";
        head = head->next;
    }
}
```

---

## Array vs Linked List
| | Array | Linked List |
|---|---|---|
| Access by index | O(1) | O(N) |
| Insert/delete at front | O(N) | O(1) |
| Memory | contiguous | scattered + pointer overhead |
| Cache locality | good | poor |

> The **dummy node** pattern (a throwaway head) simplifies insertions/deletions at the front — you'll see it constantly in LL problems.
