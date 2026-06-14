# Generate Parentheses  🟡

**Reference:** [LeetCode 22 — Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)

---

## Problem
Generate all combinations of `n` pairs of well-formed parentheses.

```
Input:  n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]
```

---

## Intuition
Track how many `(` and `)` have been used:
- Add `(` if `open < n`.
- Add `)` only if `close < open` (can't close more than we've opened — keeps it valid).
- When the string reaches length `2n`, it's a complete valid combination.

---

## Code (C++)
```cpp
void backtrack(int open, int close, int n, string cur, vector<string>& res) {
    if (cur.size() == 2 * n) { res.push_back(cur); return; }
    if (open < n)
        backtrack(open + 1, close, n, cur + '(', res);
    if (close < open)
        backtrack(open, close + 1, n, cur + ')', res);
}

vector<string> generateParenthesis(int n) {
    vector<string> res;
    backtrack(0, 0, n, "", res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(4ⁿ / √n) — the n-th Catalan number of valid strings |
| Space | O(n) recursion depth |

> The `close < open` rule prunes invalid branches early, so we only ever build valid strings — no post-filtering needed.
