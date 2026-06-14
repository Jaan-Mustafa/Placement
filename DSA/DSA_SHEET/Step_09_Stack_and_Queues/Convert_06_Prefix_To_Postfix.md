# Prefix to Postfix  🟡

**Reference:** [GeeksforGeeks — Prefix to Postfix](https://www.geeksforgeeks.org/prefix-postfix-conversion/)

---

## Problem
Convert a prefix expression into postfix notation.

```
Input:  *-A/BC-/AKL
Output: ABC/-AK/L-*
```

---

## Intuition
Scan prefix **right to left**, pushing operand strings. On an operator, pop two operands (first popped is the **left** one because of the reversed scan), and build `left + right + op` — the operator goes at the **end**. Push back; the final string is the postfix form.

---

## Code (C++)
```cpp
bool isOperand(char c) { return isalnum(c); }

string prefixToPostfix(string s) {
    stack<string> st;
    for (int i = (int)s.size() - 1; i >= 0; i--) {   // scan right to left
        char c = s[i];
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string left  = st.top(); st.pop();   // first popped = left
            string right = st.top(); st.pop();
            string expr = left + right + c;      // operator appended
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

> ⚠️ Right-to-left scan means the first pop is the left operand; concatenate as `left + right + op`.
