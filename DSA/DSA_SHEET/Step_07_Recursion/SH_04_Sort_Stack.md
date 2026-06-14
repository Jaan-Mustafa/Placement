# Sort a Stack using Recursion  🟡

**Reference:** [GeeksforGeeks — Sort a stack using recursion](https://www.geeksforgeeks.org/sort-a-stack-using-recursion/)

---

## Problem
Sort a stack (ascending, largest on top) using **only recursion** — no other data structure.

```
Input (top→bottom):  [3, 1, 4, 2]    Output: [4, 3, 2, 1]
```

---

## Intuition
Two recursive functions:
1. `sortStack`: pop the top, recursively sort the rest, then **insert** the popped element into its sorted position.
2. `insertSorted`: pop elements until the right spot for `x` is found, push `x`, then push the popped ones back.

The recursion stack acts as the temporary storage.

---

## Code (C++)
```cpp
void insertSorted(stack<int>& st, int x) {
    if (st.empty() || st.top() <= x) { st.push(x); return; }
    int top = st.top(); st.pop();
    insertSorted(st, x);            // recurse until correct spot
    st.push(top);                   // restore larger elements on top
}

void sortStack(stack<int>& st) {
    if (st.empty()) return;
    int top = st.top(); st.pop();
    sortStack(st);                  // sort the remaining stack
    insertSorted(st, top);          // insert top into sorted stack
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) — each insert may touch the whole stack |
| Space | O(N) recursion stack |
