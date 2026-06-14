# Valid Parenthesis String  🟡

**Reference:** [LeetCode 678 — Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/)

---

## Problem
A string contains `'('`, `')'`, and `'*'`. Each `'*'` can be treated as `'('`, `')'`, or an empty string `""`. Return `true` if the string can be made a valid parenthesis sequence.

```
Input:  "(*)"   Output: true
Input:  "(*))"  Output: true
Input:  ")("    Output: false
```

---

## Intuition
Track a **range** of possible open-bracket counts as we scan: `lo` (minimum opens assuming every `*` is `)` or empty) and `hi` (maximum opens assuming every `*` is `(`).

- `'('`: both `lo++`, `hi++`.
- `')'`: both `lo--`, `hi--`.
- `'*'`: `lo--` (treat as `)`) and `hi++` (treat as `(`).

If `hi` ever drops below 0, even the most generous interpretation has too many `)` → invalid. Clamp `lo` at 0 (we can never have negative real opens). At the end, the string is valid iff `lo == 0` (some interpretation closes everything exactly).

---

## Code (C++)
```cpp
bool checkValidString(string s) {
    int lo = 0, hi = 0;                // range of possible '(' counts
    for (char c : s) {
        if (c == '(') { lo++; hi++; }
        else if (c == ')') { lo--; hi--; }
        else { lo--; hi++; }           // '*' : ')' lowers lo, '(' raises hi
        if (hi < 0) return false;      // too many ')' even at best case
        if (lo < 0) lo = 0;            // can't have negative real opens
    }
    return lo == 0;                    // can close everything exactly
}
```

---

## Dry run
`"(*))"`: `(` →(1,1); `*` →(0,2); `)` →(-1→0,1); `)` →(-1→0,0). End `lo==0` → **true**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> ⚠️ Clamp `lo` to 0 but DON'T clamp `hi` — let `hi < 0` signal failure. Forgetting the clamp on `lo` causes false negatives.
