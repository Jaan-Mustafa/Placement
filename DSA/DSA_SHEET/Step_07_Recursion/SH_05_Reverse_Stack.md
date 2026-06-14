# Reverse a Stack using Recursion  🟡

**Reference:** [GeeksforGeeks — Reverse a stack using recursion](https://www.geeksforgeeks.org/reverse-a-stack-using-recursion/)

---

## Problem
Reverse a stack using only recursion — no extra data structure.

```
Input (top→bottom):  [1, 2, 3, 4]    Output (top→bottom): [4, 3, 2, 1]
```

---

## Intuition
Mirror of "sort a stack":
1. `reverseStack`: pop the top, recursively reverse the rest, then **insert the popped element at the bottom**.
2. `insertAtBottom`: recurse to empty the stack, push `x` (now at the bottom), then push everything back.

---

## Code (C++)
```cpp
void insertAtBottom(stack<int>& st, int x) {
    if (st.empty()) { st.push(x); return; }
    int top = st.top(); st.pop();
    insertAtBottom(st, x);          // go to the bottom
    st.push(top);                   // restore on the way back
}

void reverseStack(stack<int>& st) {
    if (st.empty()) return;
    int top = st.top(); st.pop();
    reverseStack(st);               // reverse the rest
    insertAtBottom(st, top);        // old top becomes the bottom
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) |
| Space | O(N) recursion stack |

> The key sub-routine `insertAtBottom` is what flips the order — each element gets pushed to the bottom in reverse pop order.
