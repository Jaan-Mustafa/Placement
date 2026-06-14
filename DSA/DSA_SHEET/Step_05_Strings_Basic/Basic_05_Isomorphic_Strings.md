# Isomorphic Strings  🟢

**Reference:** [LeetCode 205 — Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)

---

## Problem
Two strings are isomorphic if characters in `s` can be replaced to get `t`, with a **consistent one-to-one mapping** (no two characters map to the same character).

```
Input:  s = "egg", t = "add"      Output: true   (e→a, g→d)
Input:  s = "foo", t = "bar"      Output: false  (o would map to both a and r)
Input:  s = "badc", t = "baba"    Output: false  (d and c both → a/b breaks 1-1)
```

---

## Intuition
Maintain **two** mappings: `s→t` and `t→s`. For each character pair, if a mapping already exists it must match; if not, create it. Two maps enforce the bijection (one-to-one **both** ways).

---

## Code (C++)
```cpp
bool isIsomorphic(string s, string t) {
    if (s.size() != t.size()) return false;
    unordered_map<char,char> st, ts;
    for (int i = 0; i < s.size(); i++) {
        char a = s[i], b = t[i];
        if (st.count(a) && st[a] != b) return false;   // s→t conflict
        if (ts.count(b) && ts[b] != a) return false;   // t→s conflict
        st[a] = b;
        ts[b] = a;
    }
    return true;
}
```

---

## Why two maps?
A single `s→t` map allows two different `s` chars to map to the same `t` char (e.g. "badc"→"baba"). The reverse map blocks that, ensuring the mapping is a bijection.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) — at most 256 entries each |
