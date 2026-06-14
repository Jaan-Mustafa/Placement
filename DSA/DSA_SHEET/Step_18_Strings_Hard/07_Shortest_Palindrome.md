# Shortest Palindrome  🔴

**Reference:** [LeetCode 214 — Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/)

---

## Problem
Given a string `s`, you may only add characters **in front** of it. Return the shortest palindrome you can form this way.

```
Input:  "aacecaaa"   Output: "aaacecaaa"
Input:  "abcd"       Output: "dcbabcd"
```

---

## Intuition: longest palindromic *prefix* via KMP
We can only prepend characters, so the part of `s` we keep untouched at the start must already be a palindrome. We want the **longest palindromic prefix** of `s`; the remaining suffix gets mirrored and added to the front.

Trick: build the string `t = s + '#' + reverse(s)`. Compute its **LPS array** (from KMP). The last LPS value, `lps[back]`, is the length of the longest prefix of `s` that is also a suffix of `reverse(s)` — i.e. the longest palindromic prefix of `s`.

- Let `len = lps[back]`.
- The characters of `s` after that prefix, `s[len..]`, are not part of the palindrome.
- Reverse `s[len..]` and prepend it.

The `#` separator stops the LPS from matching across the boundary (preventing an overcount when `s` is long).

---

## Code (C++)
```cpp
string shortestPalindrome(string s) {
    if (s.empty()) return s;
    string rev(s.rbegin(), s.rend());
    string t = s + "#" + rev;            // '#' = char not in s, blocks overlap

    // standard LPS (failure function) over t
    int n = t.size();
    vector<int> lps(n, 0);
    int len = 0;
    for (int i = 1; i < n; ) {
        if (t[i] == t[len]) lps[i++] = ++len;
        else if (len > 0)   len = lps[len - 1];
        else                lps[i++] = 0;
    }

    int palLen = lps[n - 1];             // longest palindromic prefix length of s
    // characters of s beyond the palindromic prefix, reversed, go in front
    string toAdd = s.substr(palLen);
    reverse(toAdd.begin(), toAdd.end());
    return toAdd + s;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — building `t` and its LPS array is linear |
| Space | O(N) — the concatenated string and its LPS array |

> ⚠️ The `#` separator is essential. Without it, when `s` itself looks like a repeated pattern the LPS can match across the `s`/`reverse(s)` boundary and report a prefix longer than `s`, producing a wrong answer.
