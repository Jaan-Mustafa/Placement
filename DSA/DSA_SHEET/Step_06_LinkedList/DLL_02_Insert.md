# Insert a Node in a Doubly Linked List  🟢

**Reference:** [GeeksforGeeks — Insertion in DLL](https://www.geeksforgeeks.org/doubly-linked-list/)

---

## Problem
Insert nodes at head, tail, or before/after a given node — maintaining both `next` and `prev` links.

---

## Code (C++)
```cpp
// Insert at head
Node* insertHead(Node* head, int val) {
    Node* node = new Node(val);
    node->next = head;
    if (head) head->prev = node;
    return node;
}

// Insert after a given node
void insertAfter(Node* node, int val) {
    Node* newNode = new Node(val);
    newNode->next = node->next;
    newNode->prev = node;
    if (node->next) node->next->prev = newNode;   // fix forward node's prev
    node->next = newNode;
}
```

---

## The 4 pointers to update
When inserting `newNode` between `A` and `B`:
1. `newNode->prev = A`
2. `newNode->next = B`
3. `A->next = newNode`
4. `B->prev = newNode`

Forgetting any one breaks the bidirectional chain.

## Complexity
| | |
|---|---|
| Time | O(1) at a known position, O(N) to reach a position |
| Space | O(1) |
