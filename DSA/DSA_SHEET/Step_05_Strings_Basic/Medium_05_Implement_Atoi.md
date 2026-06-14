# Implement Atoi (String to Integer)  🟡

**Reference:** [LeetCode 8 — String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)

---

## Problem
Convert a string to a 32-bit signed integer (like C's `atoi`):
1. Skip leading whitespace.
2. Optional `+`/`-` sign.
3. Read digits until a non-digit.
4. Clamp to `[INT_MIN, INT_MAX]` on overflow.

```
Input:  "42"            Output: 42
Input:  "   -42"        Output: -42
Input:  "4193 with words"   Output: 4193
Input:  "-91283472332"  Output: -2147483648  (clamped)
```

---

## Intuition
Process the four phases in order. Build the number digit by digit, checking for overflow **before** each multiply-and-add, clamping to INT_MAX/INT_MIN as appropriate.

---

## Code (C++)
```cpp
int myAtoi(string s) {
    int i = 0, n = s.size();
    while (i < n && s[i] == ' ') i++;          // 1. skip spaces

    int sign = 1;
    if (i < n && (s[i] == '+' || s[i] == '-')) // 2. sign
        sign = (s[i++] == '-') ? -1 : 1;

    long long num = 0;
    while (i < n && isdigit(s[i])) {           // 3. read digits
        num = num * 10 + (s[i] - '0');
        if (sign == 1 && num > INT_MAX) return INT_MAX;   // 4. clamp
        if (sign == -1 && -num < INT_MIN) return INT_MIN;
        i++;
    }
    return (int)(sign * num);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Using `long long` for `num` makes overflow detection simple. With pure `int`, you'd compare against `INT_MAX/10` before multiplying.
