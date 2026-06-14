# Infix to Postfix  🟡

**Reference:** [GeeksforGeeks — Infix to Postfix](https://www.geeksforgeeks.org/convert-infix-expression-to-postfix-expression/)

---

## Problem
Convert an infix expression (operators between operands, with parentheses) into postfix (Reverse Polish), where operators follow their operands.

```
Input:  a+b*(c^d-e)^(f+g*h)-i
Output: abcd^e-fgh*+^*+i-
```

---

## Intuition
Scan left to right (Shunting-yard). Operands go straight to the output. Operators wait on a **stack** until a lower/equal-precedence operator or a closing paren forces them out. Higher precedence binds tighter, so we pop while the stack top has **≥** precedence (for `^`, which is right-associative, pop only on strictly **>**). Parentheses are pushed and later flushed up to the matching `(`.

---

## Code (C++)
```cpp
int prec(char c) {
    if (c == '^') return 3;
    if (c == '*' || c == '/') return 2;
    if (c == '+' || c == '-') return 1;
    return -1;
}

string infixToPostfix(string s) {
    stack<char> st;
    string res;
    for (char c : s) {
        if (isalnum(c)) {
            res += c;                       // operand -> output
        } else if (c == '(') {
            st.push(c);
        } else if (c == ')') {
            while (!st.empty() && st.top() != '(') {
                res += st.top(); st.pop();
            }
            if (!st.empty()) st.pop();      // discard '('
        } else {                            // operator
            // '^' is right-associative -> pop only on strictly higher prec
            while (!st.empty() && prec(st.top()) >= prec(c)
                   && !(c == '^' && st.top() == '^')) {
                res += st.top(); st.pop();
            }
            st.push(c);
        }
    }
    while (!st.empty()) { res += st.top(); st.pop(); }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each char pushed/popped once |
| Space | O(N) for the operator stack |

> ⚠️ Right-associative `^`: do **not** pop an existing `^` of equal precedence, otherwise `a^b^c` converts wrong.
