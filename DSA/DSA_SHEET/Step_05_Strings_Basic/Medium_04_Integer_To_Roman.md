# Integer to Roman  🟡

**Reference:** [LeetCode 12 — Integer to Roman](https://leetcode.com/problems/integer-to-roman/)

---

## Problem
Convert an integer (1 to 3999) to its Roman numeral.

```
Input:  58      Output: "LVIII"
Input:  1994    Output: "MCMXCIV"
```

---

## Intuition
**Greedy.** List the values (including subtractive forms 900, 400, 90, 40, 9, 4) from largest to smallest with their symbols. Repeatedly subtract the largest value that fits, appending its symbol each time.

---

## Code (C++)
```cpp
string intToRoman(int num) {
    vector<pair<int,string>> table = {
        {1000,"M"},{900,"CM"},{500,"D"},{400,"CD"},
        {100,"C"},{90,"XC"},{50,"L"},{40,"XL"},
        {10,"X"},{9,"IX"},{5,"V"},{4,"IV"},{1,"I"}
    };
    string res;
    for (auto& [val, sym] : table) {
        while (num >= val) {        // take as many of this symbol as fit
            res += sym;
            num -= val;
        }
    }
    return res;
}
```

---

## Why include 900, 400, 90, 40, 9, 4?
These subtractive forms let the same greedy loop handle cases like 4 (`IV`) and 900 (`CM`) without special logic.

## Complexity
| | |
|---|---|
| Time | O(1) — bounded by 3999 |
| Space | O(1) |
