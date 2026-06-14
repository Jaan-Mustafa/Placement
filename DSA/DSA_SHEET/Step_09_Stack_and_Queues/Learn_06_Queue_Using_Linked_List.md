# Implement Queue Using Linked List  🟡

**Reference:** [GeeksforGeeks — Queue using Linked List](https://www.geeksforgeeks.org/queue-linked-list-implementation/)

---

## Problem
Implement a FIFO queue with a singly linked list, unbounded in size. Support `push` (enqueue), `pop` (dequeue), `front`, `size`, `empty` in O(1).

```
push(10), push(20), front() -> 10, pop() -> 10, front() -> 20
```

---

## Intuition
Keep pointers to both `front` (head) and `rear` (tail). **Enqueue** appends after `rear`; **dequeue** removes from `front`. Maintaining the tail pointer keeps enqueue O(1) instead of an O(N) traversal each time.

---

## Code (C++)
```cpp
class Queue {
    struct Node {
        int data;
        Node* next;
        Node(int d) : data(d), next(nullptr) {}
    };
    Node *front, *rear;
    int sz;
public:
    Queue() : front(nullptr), rear(nullptr), sz(0) {}

    void push(int x) {
        Node* node = new Node(x);
        if (!rear) {                 // empty queue
            front = rear = node;
        } else {
            rear->next = node;       // link after tail
            rear = node;             // move tail
        }
        sz++;
    }
    int pop() {
        if (!front) return -1;
        Node* tmp = front;
        int val = tmp->data;
        front = front->next;
        if (!front) rear = nullptr;  // queue became empty
        delete tmp;
        sz--;
        return val;
    }
    int getFront() { return front ? front->data : -1; }
    int size()  { return sz; }
    bool empty() { return front == nullptr; }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(1) per operation |
| Space | O(N) for N nodes |

> ⚠️ When the last element is dequeued, reset `rear = nullptr` too — otherwise it dangles to freed memory and the next `push` breaks.
