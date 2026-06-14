# Minimum Number of Bracket Reversals  🟡

**Reference:** [GeeksforGeeks — Minimum number of bracket reversals needed to make an expression balanced](https://www.geeksforgeeks.org/minimum-number-of-bracket-reversals-needed-to-make-an-expression-balanced/)

---

## Problem
Given a string of only `{` and `}`, return the minimum number of bracket reversals required to make it balanced. Return `-1` if balancing is impossible.

```
Input:  "}}{{"    Output: 2   ->  "{}{}"
Input:  "{{{"     Output: -1  (odd length, never balanceable)
Input:  "}{"      Output: 2   ->  "{}"
```

---

## Intuition
A balanced expression must have **even length** — if the length is odd, return `-1` immediately.

Cancel every matched pair `"{}"` using a stack. After processing, the only characters left form a pattern of `m` closing brackets followed by `n` opening brackets: `} } ... } { { ... {`. These are the unmatched brackets.

- To fix `m` closing brackets we need `ceil(m/2)` reversals.
- To fix `n` opening brackets we need `ceil(n/2)` reversals.

Answer = `ceil(m/2) + ceil(n/2)`, which equals `(m+1)/2 + (n+1)/2` in integer arithmetic.

We don't even need an actual stack — just track counts of unmatched open (`open`) and unmatched close (`close`).

---

## Code (C++)
```cpp
int countMinReversals(string s) {
    int n = s.size();
    if (n % 2 != 0) return -1;            // odd length can never balance

    int open = 0;    // unmatched '{'
    int close = 0;   // unmatched '}'
    for (char c : s) {
        if (c == '{') {
            open++;                        // a new open bracket waits for a match
        } else { // c == '}'
            if (open > 0) open--;          // matches a pending open bracket
            else close++;                  // no open to match -> unmatched close
        }
    }
    // open closes need ceil(open/2), close opens need ceil(close/2)
    return (open + 1) / 2 + (close + 1) / 2;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — single pass |
| Space | O(1) — only two counters |

> ⚠️ The odd-length check must come first; otherwise the count formula silently returns a wrong (non-negative) answer for impossible inputs.
