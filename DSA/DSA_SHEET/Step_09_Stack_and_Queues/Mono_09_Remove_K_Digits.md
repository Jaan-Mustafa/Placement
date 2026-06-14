# Remove K Digits  🟡

**Reference:** [LeetCode 402 — Remove K Digits](https://leetcode.com/problems/remove-k-digits/)

---

## Problem
Given a non-negative integer as a string `num`, remove exactly `k` digits so the resulting number is the **smallest** possible. Return it without leading zeros (return `"0"` if empty).

```
Input:  num = "1432219", k = 3   Output: "1219"
Input:  num = "10200",   k = 1   Output: "200"
```

---

## Intuition
To make a number small, eliminate the first digit that is larger than the digit after it (a "descent"). A **monotonic increasing stack** does this greedily: while the top digit is greater than the incoming digit and we still have removals left, pop it. Any leftover `k` is removed from the end (the digits are already ascending). Finally strip leading zeros.

---

## Code (C++)
```cpp
string removeKdigits(string num, int k) {
    string st;                              // string used as a stack
    for (char c : num) {
        while (!st.empty() && k > 0 && st.back() > c) {
            st.pop_back();                  // remove a larger preceding digit
            k--;
        }
        st.push_back(c);
    }
    while (k > 0) { st.pop_back(); k--; }   // remove remaining from the end

    int i = 0;
    while (i < (int)st.size() && st[i] == '0') i++;   // strip leading zeros
    string res = st.substr(i);
    return res.empty() ? "0" : res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> ⚠️ Two edge cases: leftover `k` after the loop (remove from the tail) and leading zeros / empty result (return `"0"`).
