# Prefix to Infix  🟡

**Reference:** [GeeksforGeeks — Prefix to Infix](https://www.geeksforgeeks.org/prefix-infix-conversion/)

---

## Problem
Convert a prefix expression into a fully-parenthesised infix expression.

```
Input:  *+ab-cd
Output: ((a+b)*(c-d))
```

---

## Intuition
Process prefix **right to left**. Push operand strings onto a stack. On an operator, the two operands are now sitting on top with the **left operand popped first** (because of the reversed scan direction), so combine as `(left op right)` and push back.

---

## Code (C++)
```cpp
bool isOperand(char c) { return isalnum(c); }

string prefixToInfix(string s) {
    stack<string> st;
    for (int i = (int)s.size() - 1; i >= 0; i--) {   // scan right to left
        char c = s[i];
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string left  = st.top(); st.pop();   // popped first = left
            string right = st.top(); st.pop();
            string expr = "(" + left + c + right + ")";
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

> ⚠️ Because we scan from the right, the **first** value popped is the *left* operand — the opposite of the postfix-to-infix routine.
