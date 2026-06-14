# stack — LIFO

`#include <stack>`

Last-In-First-Out. A **container adaptor** — it wraps another container (default `deque`) and exposes only top-end operations. You cannot iterate or index a stack.

---

## Construction
```cpp
stack<int> st;                          // default (backed by deque)
stack<int, vector<int>> st2;            // backed by vector instead
stack<pair<int,int>> sp;                // stack of pairs

// initialize from an existing container (C++ has no init-list ctor for stack)
deque<int> d = {1,2,3};
stack<int> st3(d);                      // top = 3
```

---

## Full function list (that's all there is)
| Function | Description | Complexity |
|---|---|---|
| `st.push(x)` | Push onto top | O(1) |
| `st.emplace(args)` | Construct in place on top | O(1) |
| `st.pop()` | Remove top — **returns void** | O(1) |
| `st.top()` | Reference to top element | O(1) |
| `st.size()` | Number of elements | O(1) |
| `st.empty()` | True if empty | O(1) |
| `st.swap(st2)` | Swap two stacks | O(1) |

> There is **no** `clear()`, no iterators, no `find`. To drain, pop in a loop.

---

## Examples

### Basics
```cpp
stack<int> st;
st.push(1); st.push(2); st.push(3);   // bottom [1,2,3] top
cout << st.top();                     // 3
st.pop();                             // removes 3, top -> 2
cout << st.size();                    // 2

while (!st.empty()) {                 // drain (prints 2 1)
    cout << st.top() << " ";
    st.pop();
}
```

> ⚠️ **`pop()` returns nothing.** Always read `top()` first, then `pop()`:
> ```cpp
> int x = st.top();   // grab value
> st.pop();           // then remove
> ```

---

## DSA patterns

### 1. Balanced parentheses
```cpp
bool isValid(string s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;
            char t = st.top(); st.pop();
            if ((c == ')' && t != '(') ||
                (c == ']' && t != '[') ||
                (c == '}' && t != '{')) return false;
        }
    }
    return st.empty();
}
```

### 2. Monotonic stack — Next Greater Element
```cpp
// for each element, find the next element to its right that is greater
vector<int> nextGreater(vector<int>& a) {
    int n = a.size();
    vector<int> res(n, -1);
    stack<int> st;                       // stores INDICES, decreasing values
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[st.top()] < a[i]) {
            res[st.top()] = a[i];        // a[i] is next greater for st.top()
            st.pop();
        }
        st.push(i);
    }
    return res;
}
```

### 3. Largest rectangle in histogram (monotonic stack)
```cpp
int largestRectangle(vector<int>& h) {
    stack<int> st;                       // increasing heights (indices)
    int best = 0;
    for (int i = 0; i <= (int)h.size(); i++) {
        int cur = (i == h.size()) ? 0 : h[i];
        while (!st.empty() && h[st.top()] >= cur) {
            int height = h[st.top()]; st.pop();
            int width = st.empty() ? i : i - st.top() - 1;
            best = max(best, height * width);
        }
        st.push(i);
    }
    return best;
}
```

### 4. Iterative DFS (replaces recursion)
```cpp
void dfs(int start, vector<vector<int>>& adj) {
    vector<bool> visited(adj.size(), false);
    stack<int> st;
    st.push(start);
    while (!st.empty()) {
        int node = st.top(); st.pop();
        if (visited[node]) continue;
        visited[node] = true;
        // process node...
        for (int nbr : adj[node])
            if (!visited[nbr]) st.push(nbr);
    }
}
```

### 5. Evaluate / convert expressions
Stacks power infix→postfix conversion and postfix evaluation (operands pushed, operators pop two and push result).

---

## When to use a stack
- Matching / nesting problems (parentheses, tags, valid sequences).
- **Monotonic stack:** next/previous greater/smaller element, histogram, stock span, trapping rain water.
- Undo operations, backtracking, iterative DFS.
- Expression parsing and evaluation.

## Gotchas
- ⚠️ No iteration/indexing — if you need to inspect the middle, use a `vector` as a manual stack (`push_back`/`pop_back`/`back`).
- ⚠️ `top()` / `pop()` on an empty stack is **UB** — always check `empty()` first.
- Backing with `vector` (`stack<int,vector<int>>`) can be slightly faster than the default `deque` for plain ints.
