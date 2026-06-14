# Remove Outermost Parentheses  🟢

**Reference:** [LeetCode 1021 — Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/)

---

## Problem
A valid parentheses string is a concatenation of **primitive** valid strings. Remove the outermost parentheses of every primitive.

```
Input:  "(()())(())"    Output: "()()()"
Input:  "(()())(())(()(()))"    Output: "()()()()(())"
```

---

## Intuition
Track the nesting `depth`. An opening `(` is part of output only if depth > 0 **before** incrementing (i.e. it's not an outermost open). A closing `)` is output only if depth > 1 before decrementing. This skips exactly the outer layer of each primitive.

---

## Code (C++)
```cpp
string removeOuterParentheses(string s) {
    string res;
    int depth = 0;
    for (char c : s) {
        if (c == '(') {
            if (depth > 0) res += c;     // not an outer '('
            depth++;
        } else {
            depth--;
            if (depth > 0) res += c;     // not an outer ')'
        }
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) for the output |
