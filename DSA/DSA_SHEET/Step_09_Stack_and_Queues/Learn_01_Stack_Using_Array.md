# Implement Stack Using Array  🟢

**Reference:** [GeeksforGeeks — Implement Stack using Array](https://www.geeksforgeeks.org/implement-stack-using-array/) · [LeetCode 232/225 (related)](https://leetcode.com/problems/implement-stack-using-queues/)

---

## Problem
Implement a LIFO stack backed by a fixed-size array supporting `push`, `pop`, `top`, `size`, and `empty` — all in O(1).

```
push(10), push(20), top() -> 20, pop() -> 20, top() -> 10
```

---

## Intuition
A stack only grows/shrinks at one end. Keep an integer `topIndex` that points at the last inserted element. `push` writes at `++topIndex`, `pop` returns `arr[topIndex--]`. The whole structure is O(1) per op because we never shift elements.

---

## Code (C++)
```cpp
class Stack {
    int* arr;
    int capacity;
    int topIndex;          // index of the current top, -1 when empty
public:
    Stack(int cap = 1000) {
        capacity = cap;
        arr = new int[capacity];
        topIndex = -1;
    }
    ~Stack() { delete[] arr; }

    void push(int x) {
        if (topIndex == capacity - 1) return;   // overflow guard
        arr[++topIndex] = x;
    }
    int pop() {
        if (topIndex == -1) return -1;           // underflow guard
        return arr[topIndex--];
    }
    int top() {
        if (topIndex == -1) return -1;
        return arr[topIndex];
    }
    int size()  { return topIndex + 1; }
    bool empty() { return topIndex == -1; }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(1) per operation |
| Space | O(capacity) |

> ⚠️ Always check overflow (`topIndex == capacity-1`) and underflow (`topIndex == -1`) before touching `arr`, otherwise you index out of bounds.
