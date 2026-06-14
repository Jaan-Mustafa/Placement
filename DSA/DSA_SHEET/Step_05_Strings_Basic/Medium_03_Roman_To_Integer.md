# Roman to Integer  🟡

**Reference:** [LeetCode 13 — Roman to Integer](https://leetcode.com/problems/roman-to-integer/)

---

## Problem
Convert a Roman numeral to an integer.

```
Input:  "III"       Output: 3
Input:  "LVIII"     Output: 58
Input:  "MCMXCIV"   Output: 1994
```

---

## Intuition
Map each symbol to its value. Normally values add up; but when a smaller value precedes a larger one (e.g. `IV` = 4, `IX` = 9), it is **subtracted**. So scan left to right: if the current symbol's value is less than the next symbol's value, subtract it; otherwise add it.

---

## Code (C++)
```cpp
int romanToInt(string s) {
    unordered_map<char,int> val = {
        {'I',1},{'V',5},{'X',10},{'L',50},
        {'C',100},{'D',500},{'M',1000}
    };
    int total = 0;
    for (int i = 0; i < s.size(); i++) {
        if (i + 1 < s.size() && val[s[i]] < val[s[i+1]])
            total -= val[s[i]];     // subtractive case (IV, IX, XL...)
        else
            total += val[s[i]];
    }
    return total;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) — fixed map of 7 entries |
