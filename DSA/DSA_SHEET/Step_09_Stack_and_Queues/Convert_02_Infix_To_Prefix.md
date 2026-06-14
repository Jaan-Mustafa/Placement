# Infix to Prefix  🟡

**Reference:** [GeeksforGeeks — Infix to Prefix](https://www.geeksforgeeks.org/convert-infix-prefix-notation/)

---

## Problem
Convert an infix expression into prefix (Polish) notation, where each operator precedes its operands.

```
Input:  (a-b/c)*(a/k-l)
Output: *-a/bc-/akl
```

---

## Intuition
Prefix is the mirror of postfix. The trick: **reverse** the string (swapping `(` with `)`), run the infix→postfix routine, then **reverse the result**. The only subtlety is associativity flips on the reversed string — so for `^` (right-associative) we now pop on `>=`, while for left-associative operators we pop only on strictly `>` to preserve order.

---

## Code (C++)
```cpp
int prec(char c) {
    if (c == '^') return 3;
    if (c == '*' || c == '/') return 2;
    if (c == '+' || c == '-') return 1;
    return -1;
}

string infixToPostfixForPrefix(string s) {
    stack<char> st;
    string res;
    for (char c : s) {
        if (isalnum(c)) res += c;
        else if (c == '(') st.push(c);
        else if (c == ')') {
            while (!st.empty() && st.top() != '(') { res += st.top(); st.pop(); }
            if (!st.empty()) st.pop();
        } else {
            // after reversal, left-assoc ops pop only on strictly higher prec
            while (!st.empty() && (prec(st.top()) > prec(c)
                   || (prec(st.top()) == prec(c) && c == '^'))) {
                res += st.top(); st.pop();
            }
            st.push(c);
        }
    }
    while (!st.empty()) { res += st.top(); st.pop(); }
    return res;
}

string infixToPrefix(string s) {
    reverse(s.begin(), s.end());
    for (char &c : s) {                    // swap brackets
        if (c == '(') c = ')';
        else if (c == ')') c = '(';
    }
    string post = infixToPostfixForPrefix(s);
    reverse(post.begin(), post.end());     // reverse back -> prefix
    return post;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> ⚠️ Two things flip when you reverse: the brackets must be swapped, and the precedence comparison (`>` vs `>=`) inverts versus the plain infix→postfix routine.
