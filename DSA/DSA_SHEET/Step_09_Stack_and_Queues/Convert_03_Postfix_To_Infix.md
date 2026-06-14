# Postfix to Infix  🟡

**Reference:** [GeeksforGeeks — Postfix to Infix](https://www.geeksforgeeks.org/postfix-to-infix/)

---

## Problem
Convert a postfix expression into a fully-parenthesised infix expression.

```
Input:  ab*c+
Output: ((a*b)+c)
```

---

## Intuition
Scan left to right pushing **operand strings** onto a stack. On an operator, pop the two most recent operands (the second pop is the left operand), wrap them as `(left op right)`, and push the combined string back. The single string left at the end is the answer.

---

## Code (C++)
```cpp
bool isOperand(char c) { return isalnum(c); }

string postfixToInfix(string s) {
    stack<string> st;
    for (char c : s) {
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string right = st.top(); st.pop();   // popped first = right
            string left  = st.top(); st.pop();
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
| Time | O(N) operations; O(N) total string building |
| Space | O(N) for the stack of partial expressions |

> ⚠️ Order matters: the **first** value popped is the right operand, the second is the left. Swapping them breaks non-commutative operators like `-` and `/`.
