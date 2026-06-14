# Letter Combinations of a Phone Number  🟡

**Reference:** [LeetCode 17 — Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

---

## Problem
Given digits 2–9, return all letter combinations the number could represent (like old phone keypads).

```
Input:  "23"
Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

---

## Intuition
Map each digit to its letters. Recurse position by position: at digit index `i`, try each letter of that digit, append it, and recurse to `i+1`. At the end of the digit string, record the built combination. This is a Cartesian product via recursion.

---

## Code (C++)
```cpp
void solve(int i, string& digits, string cur,
           vector<string>& map, vector<string>& res) {
    if (i == digits.size()) {
        if (!cur.empty()) res.push_back(cur);
        return;
    }
    string letters = map[digits[i] - '0'];
    for (char c : letters)
        solve(i + 1, digits, cur + c, map, res);
}

vector<string> letterCombinations(string digits) {
    if (digits.empty()) return {};
    vector<string> map = {"", "", "abc", "def", "ghi", "jkl",
                          "mno", "pqrs", "tuv", "wxyz"};
    vector<string> res;
    solve(0, digits, "", map, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(4ⁿ · n) — up to 4 letters per digit |
| Space | O(n) recursion depth |

> Each level of recursion fixes one digit's letter; the branching factor is that digit's number of letters (3 or 4).
