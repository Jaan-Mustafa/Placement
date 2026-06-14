# Expression Add Operators  🔴

**Reference:** [LeetCode 282 — Expression Add Operators](https://leetcode.com/problems/expression-add-operators/)

---

## Problem
Given a digit string `num` and a `target`, insert the binary operators `+`, `-`, `*` (or nothing) between digits so the resulting expression evaluates to `target`. Return all valid expressions.

```
Input:  num = "123", target = 6
Output: ["1*2*3", "1+2+3"]
```

---

## Intuition
Backtrack over split points: take the next operand `num[start..i]`, then choose an operator. The trap is `*`, which binds tighter than `+/-`. Track the running `value` and the `prev` operand: for multiplication, subtract `prev` and add `prev * cur`, so chained products fold correctly without re-parsing.

---

## Code (C++)
```cpp
void solve(const string& num, long target, int start, string expr,
           long value, long prev, vector<string>& res) {
    if (start == (int)num.size()) {
        if (value == target) res.push_back(expr);
        return;
    }
    for (int i = start; i < (int)num.size(); i++) {
        if (i > start && num[start] == '0') break;        // ⚠️ no leading zeros
        string part = num.substr(start, i - start + 1);
        long cur = stol(part);

        if (start == 0) {                                  // first operand: no op
            solve(num, target, i + 1, part, cur, cur, res);
        } else {
            // addition
            solve(num, target, i + 1, expr + "+" + part, value + cur, cur, res);
            // subtraction
            solve(num, target, i + 1, expr + "-" + part, value - cur, -cur, res);
            // multiplication: undo prev, apply prev*cur (precedence fix)
            solve(num, target, i + 1, expr + "*" + part,
                  value - prev + prev * cur, prev * cur, res);
        }
    }
}

vector<string> addOperators(string num, int target) {
    vector<string> res;
    solve(num, (long)target, 0, "", 0, 0, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(4^N) — at each gap: extend operand or pick one of 3 operators |
| Space | O(N) recursion depth + expression string |

> ⚠️ Two pitfalls: skip operands with leading zeros (`"05"` is invalid), and handle `*` precedence via `value - prev + prev*cur`. Use `long` to avoid overflow on big operands.
