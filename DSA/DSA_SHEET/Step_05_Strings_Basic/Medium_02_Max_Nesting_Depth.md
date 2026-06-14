# Maximum Nesting Depth of Parentheses  🟡

**Reference:** [LeetCode 1614 — Maximum Nesting Depth of the Parentheses](https://leetcode.com/problems/maximum-nesting-depth-of-the-parentheses/)

---

## Problem
Given a valid parentheses string (may contain digits/operators), return the maximum nesting depth.

```
Input:  "(1+(2*3)+((8)/4))+1"    Output: 3
Input:  "1+(2*3)/(2-1)"          Output: 1
```

---

## Intuition
Track a running `depth`: increment on `(`, decrement on `)`. The answer is the maximum value `depth` ever reaches. No stack needed since the string is guaranteed valid.

---

## Code (C++)
```cpp
int maxDepth(string s) {
    int depth = 0, best = 0;
    for (char c : s) {
        if (c == '(') { depth++; best = max(best, depth); }
        else if (c == ')') depth--;
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
