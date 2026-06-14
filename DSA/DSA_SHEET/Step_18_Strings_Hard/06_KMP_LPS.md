# KMP Algorithm / LPS Array  🔴

**Reference:** [GeeksforGeeks — KMP Algorithm for Pattern Searching](https://www.geeksforgeeks.org/kmp-algorithm-for-pattern-searching/) · [LeetCode 28 — Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/)

---

## Problem
Find occurrences of pattern `pat` (length `m`) in text `txt` (length `n`) in O(n + m), without re-examining characters of the text.

```
txt = "aabaacaadaabaaba", pat = "aaba"   ->  indices 0, 9, 12
```

---

## Intuition: the LPS / failure function
The heart of KMP is the **LPS array** (Longest Proper Prefix which is also a Suffix). `lps[i]` = length of the longest proper prefix of `pat[0..i]` that is also a suffix of `pat[0..i]`. ("Proper" = not the whole substring.)

```
pat = "a a b a a"
lps = "0 1 0 1 2"
```

**Why it helps:** while matching, if a mismatch happens at pattern position `j`, we already know `pat[0..j-1]` matched the text. The LPS tells us the longest prefix that is also a suffix of that matched portion — so we can resume comparing at `pat[lps[j-1]]` **without moving the text pointer backward**. No text character is ever re-examined, giving linear time.

**Building LPS** uses the same idea on the pattern against itself: a `len` pointer tracks the current longest prefix-suffix; on a mismatch it falls back to `lps[len-1]` instead of resetting to 0.

---

## Code (C++)
```cpp
vector<int> buildLPS(const string& pat) {
    int m = pat.size();
    vector<int> lps(m, 0);
    int len = 0;                       // length of the current longest prefix-suffix
    for (int i = 1; i < m; ) {
        if (pat[i] == pat[len]) {
            lps[i++] = ++len;          // extend the prefix-suffix
        } else if (len > 0) {
            len = lps[len - 1];        // fall back, but DON'T advance i
        } else {
            lps[i++] = 0;              // no prefix-suffix here
        }
    }
    return lps;
}

vector<int> kmpSearch(const string& txt, const string& pat) {
    int n = txt.size(), m = pat.size();
    vector<int> result;
    if (m == 0 || m > n) return result;
    vector<int> lps = buildLPS(pat);

    int i = 0, j = 0;                  // i -> txt, j -> pat
    while (i < n) {
        if (txt[i] == pat[j]) {
            i++; j++;
            if (j == m) {              // full pattern matched
                result.push_back(i - m);
                j = lps[j - 1];        // continue searching for overlaps
            }
        } else if (j > 0) {
            j = lps[j - 1];            // reuse known prefix-suffix; i stays put
        } else {
            i++;                       // no progress in pattern, advance text
        }
    }
    return result;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n + m) — O(m) to build LPS, O(n) to scan; text pointer never retreats |
| Space | O(m) — the LPS array |

> ⚠️ On a mismatch the fallback is always `lps[len-1]` / `lps[j-1]` — never reset to 0 directly, or you lose the partial match and break linearity. When building LPS, do **not** advance `i` on the fallback branch.
