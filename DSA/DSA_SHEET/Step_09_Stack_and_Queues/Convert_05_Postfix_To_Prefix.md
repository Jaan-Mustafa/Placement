# Postfix to Prefix  🟡

**Reference:** [GeeksforGeeks — Postfix to Prefix](https://www.geeksforgeeks.org/postfix-to-prefix-conversion/)

---

## Problem
Convert a postfix expression into prefix notation.

```
Input:  ABC/-AK/L-*
Output: *-A/BC-/AKL
```

---

## Intuition
Scan postfix **left to right**, pushing operand strings. On an operator, pop two operands (first popped is the right one) and build the prefix string `op + left + right` — the operator goes in **front**. Push the result back. The last remaining string is the prefix expression.

---

## Code (C++)
```cpp
bool isOperand(char c) { return isalnum(c); }

string postfixToPrefix(string s) {
    stack<string> st;
    for (char c : s) {
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string right = st.top(); st.pop();   // first popped = right
            string left  = st.top(); st.pop();
            string expr = c + left + right;      // operator prefixed
            st.push(expr);
        }
    }
    return st.top();
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> ⚠️ Concatenation order is `op + left + right`. The first value popped is the right operand — get this backwards and `-`/`/` come out wrong.
