# Recursive Implementation of atoi()  🟡

**Reference:** [LeetCode 8 — String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/) · [GeeksforGeeks](https://www.geeksforgeeks.org/recursive-implementation-of-atoi/)

---

## Problem
Convert a numeric string to an integer using recursion.

```
Input:  "123"     Output: 123
Input:  "-45"     Output: -45
```

---

## Intuition
A number `"123"` equals `atoi("12") * 10 + 3`. So recurse on all-but-last digit, multiply by 10, and add the last digit. Handle the sign at the top level before recursing on the digit portion.

---

## Code (C++)
```cpp
int atoiHelper(const string& s, int n) {
    if (n == 0) return 0;                         // base case
    int last = s[n - 1] - '0';                    // last digit
    return atoiHelper(s, n - 1) * 10 + last;      // build from the left
}

int myAtoi(string s) {
    int sign = 1, start = 0;
    if (!s.empty() && (s[0] == '-' || s[0] == '+')) {
        sign = (s[0] == '-') ? -1 : 1;
        start = 1;
    }
    return sign * atoiHelper(s.substr(start), s.size() - start);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |

> The recurrence `value(n) = value(n-1) * 10 + digit` is the recursive analogue of the iterative `num = num*10 + digit` loop.
