# Count and Say  🟡

**Reference:** [LeetCode 38 — Count and Say](https://leetcode.com/problems/count-and-say/)

---

## Problem
The count-and-say sequence is defined recursively: `countAndSay(1) = "1"`, and `countAndSay(n)` is the **run-length encoding** (read out loud) of `countAndSay(n-1)`. Return the n-th term.

```
1:  "1"
2:  "11"      (one 1)
3:  "21"      (two 1s)
4:  "1211"    (one 2, one 1)
5:  "111221"  (one 1, one 2, two 1s)
```

---

## Intuition
Start from the base case `"1"`. To build term `i` from term `i-1`, scan the previous string and group consecutive equal digits. For each group, append `count` followed by the `digit`. That run-length encoding becomes the next term.

There is no closed-form shortcut — you must generate every term up to `n`.

---

## Code (C++)
```cpp
string countAndSay(int n) {
    string s = "1";                          // term 1
    for (int term = 2; term <= n; term++) {
        string next;
        int i = 0, len = s.size();
        while (i < len) {
            int j = i;
            while (j < len && s[j] == s[i]) j++;  // extend the run of equal digits
            int count = j - i;
            next += to_string(count);        // append the count ...
            next += s[i];                    // ... then the digit being counted
            i = j;                           // jump to the next run
        }
        s = move(next);                      // this term becomes input for the next
    }
    return s;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · L) — N terms, where L is the (growing) length of the current term |
| Space | O(L) — to hold the current and next term |

> ⚠️ The digits in the *output* are the count and the symbol, never an arithmetic operation — `"3"` followed by `"3"` becomes `"23"` (two 3s), not `"6"`.
