# Largest Odd Number in a String  🟢

**Reference:** [LeetCode 1903 — Largest Odd Number in String](https://leetcode.com/problems/largest-odd-number-in-string/)

---

## Problem
Given a numeric string, return the largest-valued **odd** number that is a non-empty prefix of it (you may only drop trailing digits). Return "" if none.

```
Input:  "52"     Output: "5"
Input:  "4206"   Output: ""    (no odd prefix)
Input:  "35427"  Output: "35427"
```

---

## Intuition
A number is odd iff its **last digit** is odd. To keep the value largest, we want the longest prefix ending in an odd digit. Scan from the right for the first odd digit; the prefix up to and including it is the answer.

---

## Code (C++)
```cpp
string largestOddNumber(string num) {
    for (int i = num.size() - 1; i >= 0; i--) {
        if ((num[i] - '0') % 2 == 1)        // odd digit found
            return num.substr(0, i + 1);    // prefix up to here
    }
    return "";                              // no odd digit at all
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) (excluding the returned substring) |

> Only the rightmost odd digit's position matters — everything left of it is kept to maximize the value.
