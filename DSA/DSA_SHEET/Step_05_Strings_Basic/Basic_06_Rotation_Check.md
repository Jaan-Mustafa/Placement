# Check if Strings are Rotations of Each Other  🟢

**Reference:** [LeetCode 796 — Rotate String](https://leetcode.com/problems/rotate-string/) · [GeeksforGeeks](https://www.geeksforgeeks.org/a-program-to-check-if-strings-are-rotations-of-each-other/)

---

## Problem
Check whether string `b` is a rotation of string `a`.

```
Input:  a = "abcde", b = "cdeab"    Output: true
Input:  a = "abcde", b = "abced"    Output: false
```

---

## Intuition
Every rotation of `a` is a substring of `a + a` (a concatenated with itself). For example, `"abcde" + "abcde" = "abcdeabcde"` contains all rotations: `bcdea`, `cdeab`, etc. So `b` is a rotation iff it's the same length as `a` **and** a substring of `a + a`.

---

## Code (C++)
```cpp
bool rotateString(string a, string b) {
    if (a.size() != b.size()) return false;
    string doubled = a + a;
    return doubled.find(b) != string::npos;     // b appears in a+a
}
```

---

## Why `a + a`?
Rotating `a` by `k` gives `a[k:] + a[:k]`, which is exactly the window `a+a[k .. k+n)`. Scanning all windows of `a+a` covers every possible rotation.

## Complexity
| | |
|---|---|
| Time | O(N²) with naive `find` (O(N) with KMP) |
| Space | O(N) for the doubled string |
