# Introduction to Doubly Linked List  🟢

**Reference:** [GeeksforGeeks — Doubly Linked List](https://www.geeksforgeeks.org/doubly-linked-list/)

---

## What is a Doubly Linked List (DLL)?
Each node has **two** pointers: `next` and `prev`. This allows traversal in both directions and O(1) deletion when you hold a node (no need to find the predecessor).

```
null ← [1] ⇄ [2] ⇄ [3] → null
```

---

## Node definition
```cpp
struct Node {
    int val;
    Node* next;
    Node* prev;
    Node(int x) : val(x), next(nullptr), prev(nullptr) {}
};
```

---

## Build a DLL from an array
```cpp
Node* buildDLL(vector<int>& arr) {
    if (arr.empty()) return nullptr;
    Node* head = new Node(arr[0]);
    Node* prev = head;
    for (int i = 1; i < arr.size(); i++) {
        Node* node = new Node(arr[i]);
        prev->next = node;
        node->prev = prev;      // link backward too
        prev = node;
    }
    return head;
}
```

---

## DLL vs Singly LL
| | Singly LL | Doubly LL |
|---|---|---|
| Backward traversal | ❌ | ✅ |
| Delete a known node | O(N) (need prev) | O(1) |
| Memory per node | 1 pointer | 2 pointers |

> DLLs power LRU caches and browser history (back/forward) thanks to O(1) deletion at both ends.
