# Z-Function  🔴

**Reference:** [GeeksforGeeks — Z algorithm (Linear time pattern searching)](https://www.geeksforgeeks.org/z-algorithm-linear-time-pattern-searching-algorithm/) · [cp-algorithms — Z-function](https://cp-algorithms.com/string/z-function.html)

---

## Problem
For a string `s`, the Z-array `z` is defined so that `z[i]` = length of the longest substring starting at `i` that is also a **prefix** of `s`. By convention `z[0] = 0` (or `n`). Compute the whole Z-array in O(N).

```
s = "aabxaabxcaabxaabxay"
z = "0 1 0 0 4 1 0 0 0 7 1 0 0 4 1 0 0 1 0"
```

`z[4] = 4` because `"aabx"` starting at index 4 matches the prefix `"aabx"`.

---

## Intuition: the Z-box
We maintain a window `[l, r]` — the **Z-box** — which is the rightmost-reaching prefix-match found so far (`s[l..r]` equals `s[0..r-l]`).

For each index `i`:
- **If `i > r`** (outside the box), we have no information — compare characters from scratch starting at `i`, and open a new box.
- **If `i <= r`** (inside the box), the character `s[i]` mirrors `s[i-l]` inside the prefix. So `z[i]` starts as `min(r - i + 1, z[i - l])`:
  - `z[i-l]` is the already-known match length at the mirror position,
  - `r - i + 1` clamps it to the part still inside the box (beyond `r` we know nothing yet).
  Then we keep extending with direct comparison past `r` if possible.

Whenever a match extends beyond `r`, we slide the box forward (`l = i`, `r = i + z[i] - 1`). Because `r` only ever moves forward, the total character comparisons are O(N).

**Pattern search:** build `z` for `pattern + '#' + text`; any position where `z[i] == pattern.length()` marks a match.

---

## Code (C++)
```cpp
vector<int> zFunction(const string& s) {
    int n = s.size();
    vector<int> z(n, 0);              // z[0] left as 0 by convention
    int l = 0, r = 0;                 // current Z-box [l, r]
    for (int i = 1; i < n; i++) {
        if (i < r) {
            // i is inside the box: reuse mirror value, clamped to the box
            z[i] = min(r - i, z[i - l]);
        }
        // try to extend the match past whatever we already have
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        // if we reached further right than the current box, move the box
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — `r` is monotonically non-decreasing, so comparisons total O(N) |
| Space | O(N) — the Z-array |

> ⚠️ Use `r - i` (not `r - i + 1`) when the box is stored as the half-open style above (`r` = one past the last matched index). Mixing the two conventions is the classic off-by-one bug. For pattern matching, pick a separator (`#`) that cannot appear in either string.
