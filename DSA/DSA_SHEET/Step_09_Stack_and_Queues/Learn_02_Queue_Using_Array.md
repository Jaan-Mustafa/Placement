# Implement Queue Using Array  🟢

**Reference:** [GeeksforGeeks — Queue using Array](https://www.geeksforgeeks.org/queue-using-array-implementation/)

---

## Problem
Implement a FIFO queue backed by an array supporting `push` (enqueue), `pop` (dequeue), `front`, `size`, and `empty` — all in O(1).

```
push(10), push(20), front() -> 10, pop() -> 10, front() -> 20
```

---

## Intuition
A naive queue wastes space once `front` advances. Use a **circular array**: keep `front`, `rear`, and `count`. Advance indices with modulo `capacity` so freed slots at the start are reused. `count` cleanly distinguishes full from empty.

---

## Code (C++)
```cpp
class Queue {
    int* arr;
    int capacity, front, rear, count;
public:
    Queue(int cap = 1000) {
        capacity = cap;
        arr = new int[capacity];
        front = 0; rear = -1; count = 0;
    }
    ~Queue() { delete[] arr; }

    void push(int x) {
        if (count == capacity) return;           // full
        rear = (rear + 1) % capacity;            // wrap around
        arr[rear] = x;
        count++;
    }
    int pop() {
        if (count == 0) return -1;               // empty
        int val = arr[front];
        front = (front + 1) % capacity;          // wrap around
        count--;
        return val;
    }
    int getFront() { return count == 0 ? -1 : arr[front]; }
    int size()  { return count; }
    bool empty() { return count == 0; }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(1) per operation |
| Space | O(capacity) |

> ⚠️ Use modulo arithmetic on both `front` and `rear`. Tracking `count` avoids the classic "full vs empty look identical" ambiguity of circular buffers.
