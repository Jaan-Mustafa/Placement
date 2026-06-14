# Rabin-Karp Algorithm  🔴

**Reference:** [GeeksforGeeks — Rabin-Karp Algorithm for Pattern Searching](https://www.geeksforgeeks.org/rabin-karp-algorithm-for-pattern-searching/) · [LeetCode 28 — Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/)

---

## Problem
Find all starting indices where pattern `pat` (length `m`) occurs in text `txt` (length `n`).

```
txt = "abcabcabd", pat = "abd"   ->  index 6
txt = "aaaaa",     pat = "aa"     ->  indices 0,1,2,3
```

---

## Intuition: rolling hash + verify
Naive matching re-hashes every window of length `m` in O(m), giving O(n·m). Rabin-Karp computes the hash of the pattern once, then **slides a window** over the text updating its hash in O(1) per step.

The key is the **rolling update**: when the window moves one character right, we

1. subtract the contribution of the outgoing (leftmost) character,
2. shift the remaining hash by multiplying by base `p`,
3. add the incoming (rightmost) character.

When a window's hash equals the pattern's hash, the strings *might* be equal (a hash collision is possible), so we **verify with a direct character comparison**. With a good modulus, verification rarely triggers falsely, so the expected cost stays O(n + m).

---

## Code (C++)
```cpp
vector<int> rabinKarp(const string& txt, const string& pat) {
    int n = txt.size(), m = pat.size();
    vector<int> result;
    if (m > n || m == 0) return result;

    const long long p = 256;          // base (covers extended ASCII)
    const long long mod = 1e9 + 7;    // large prime to limit collisions

    long long patHash = 0, winHash = 0, highPow = 1;
    // highPow = p^(m-1) mod mod : weight of the leftmost char in the window
    for (int i = 0; i < m - 1; i++) highPow = (highPow * p) % mod;

    // initial hash of pattern and of first window of txt
    for (int i = 0; i < m; i++) {
        patHash = (patHash * p + pat[i]) % mod;
        winHash = (winHash * p + txt[i]) % mod;
    }

    for (int i = 0; i <= n - m; i++) {
        if (winHash == patHash) {
            // verify to rule out a hash collision
            if (txt.compare(i, m, pat) == 0) result.push_back(i);
        }
        // roll the window forward by one character
        if (i < n - m) {
            // remove outgoing char, shift left, add incoming char
            winHash = ((winHash - txt[i] * highPow % mod + mod) % mod * p
                       + txt[i + m]) % mod;
        }
    }
    return result;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n + m) expected; O(n·m) worst case (many collisions / forced verification) |
| Space | O(1) extra (excluding the output list) |

> ⚠️ Always verify a hash match with a real comparison — equal hashes do not guarantee equal strings. The `+ mod` after the subtraction prevents the intermediate value from going negative.
