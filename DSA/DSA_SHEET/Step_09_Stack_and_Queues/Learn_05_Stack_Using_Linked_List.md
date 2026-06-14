# Implement Stack Using Linked List  🟡

**Reference:** [GeeksforGeeks — Stack using Linked List](https://www.geeksforgeeks.org/implement-a-stack-using-singly-linked-list/)

---

## Problem
Implement a stack with a singly linked list so there is no fixed capacity. Support `push`, `pop`, `top`, `size`, `empty` in O(1).

```
push(10), push(20), top() -> 20, pop() -> 20, size() -> 1
```

---

## Intuition
Insert and delete at the **head** of the list — both are O(1) pointer updates and the head naturally behaves as the stack top. No resizing is ever needed because nodes are allocated on demand.

---

## Code (C++)
```cpp
class Stack {
    struct Node {
        int data;
        Node* next;
        Node(int d) : data(d), next(nullptr) {}
    };
    Node* head;          // head == top of stack
    int sz;
public:
    Stack() : head(nullptr), sz(0) {}

    void push(int x) {
        Node* node = new Node(x);
        node->next = head;     // new node points to old top
        head = node;           // becomes new top
        sz++;
    }
    int pop() {
        if (!head) return -1;
        Node* tmp = head;
        int val = tmp->data;
        head = head->next;
        delete tmp;            // free memory
        sz--;
        return val;
    }
    int top()   { return head ? head->data : -1; }
    int size()  { return sz; }
    bool empty() { return head == nullptr; }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(1) per operation |
| Space | O(N) for N nodes |

> ⚠️ `delete` the popped node to avoid memory leaks, and read its data before deleting.
