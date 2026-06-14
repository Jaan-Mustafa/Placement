# Implement Queue Using Stack  🟡

**Reference:** [LeetCode 232 — Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)

---

## Problem
Build a FIFO queue using only stack operations (`push`, `pop`/top, `empty`).

```
push(1), push(2), peek() -> 1, pop() -> 1, empty() -> false
```

---

## Intuition
Use **two stacks**, `input` and `output`. New elements go onto `input`. When we need the front, if `output` is empty we pour all of `input` into `output`, reversing the order — so the oldest element ends up on top of `output`. This **amortised O(1)** scheme moves each element at most once between stacks.

---

## Code (C++)
```cpp
class MyQueue {
    stack<int> input, output;
    void transfer() {                  // move only when output is empty
        if (output.empty())
            while (!input.empty()) {
                output.push(input.top());
                input.pop();
            }
    }
public:
    void push(int x) { input.push(x); }

    int pop() {
        transfer();
        int val = output.top();
        output.pop();
        return val;
    }
    int peek()  { transfer(); return output.top(); }
    bool empty() { return input.empty() && output.empty(); }
};
```

---

## Complexity
| | |
|---|---|
| Time | push O(1); pop/peek amortised O(1) |
| Space | O(N) |

> ⚠️ Only transfer when `output` is empty — refilling early would scramble the FIFO order.
