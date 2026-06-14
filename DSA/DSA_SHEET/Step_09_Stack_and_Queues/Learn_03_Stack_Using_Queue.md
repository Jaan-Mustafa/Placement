# Implement Stack Using Queue  🟡

**Reference:** [LeetCode 225 — Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/)

---

## Problem
Build a LIFO stack using only standard queue operations (`push`/enqueue at back, `pop`/dequeue from front, `front`, `size`, `empty`).

```
push(1), push(2), top() -> 2, pop() -> 2, empty() -> false
```

---

## Intuition
Use a **single queue** and make `push` costly. After enqueuing the new element at the back, rotate the queue: dequeue and re-enqueue every *older* element behind it. Now the newest element sits at the front, so `pop`/`top` are just the queue's front operations — giving true LIFO behaviour.

---

## Code (C++)
```cpp
class MyStack {
    queue<int> q;
public:
    void push(int x) {
        q.push(x);
        // rotate so the new element comes to the front
        for (int i = 0; i < (int)q.size() - 1; i++) {
            q.push(q.front());   // move front to back
            q.pop();
        }
    }
    int pop() {
        int val = q.front();
        q.pop();
        return val;
    }
    int top()   { return q.front(); }
    bool empty() { return q.empty(); }
};
```

---

## Complexity
| | |
|---|---|
| Time | push O(N), pop/top O(1) |
| Space | O(N) |

> The single-queue trick front-loads all the work into `push`. A two-queue variant exists but the one-queue rotation is cleaner. Capture `q.size()` before the loop since it changes as you push.
