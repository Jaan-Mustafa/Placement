# Hashing in Strings (Theory)  🟡

**Reference:** [GeeksforGeeks — String hashing using Polynomial rolling hash function](https://www.geeksforgeeks.org/string-hashing-using-polynomial-rolling-hash-function/)

---

## Problem
Understand how to map a string to an integer (a "hash") so that two strings can be compared in **O(1)** instead of O(N), and how to compute the hash of any substring in O(1) after O(N) preprocessing. This underpins Rabin-Karp, duplicate-substring detection, and many competitive problems.

```
"abc"  ->  some integer H
"abc"  ->  the SAME integer H        (equal strings hash equally)
"abd"  ->  a DIFFERENT integer       (with very high probability)
```

---

## Intuition: polynomial rolling hash
Treat the string as a number in base `p` (a base larger than the alphabet, e.g. 31 for lowercase, 53 for mixed case), taken modulo a large prime `m`:

```
hash(s) = ( s[0]·p^0 + s[1]·p^1 + ... + s[n-1]·p^(n-1) ) mod m
```

**Prefix hashes** let us query any substring. Define `H[i]` = hash of prefix `s[0..i-1]`. Then the hash of substring `s[l..r]` is:

```
sub(l, r) = ( H[r+1] - H[l] ) · p^(-l)   mod m
```

To avoid the modular inverse, a common trick is to compare two substrings of equal length by scaling, or to store prefix hashes and multiply by powers of `p` to align them.

**Why it works:** equal strings always produce equal hashes. Different strings *almost always* produce different hashes; a rare coincidence is a **collision**.

**Reducing collisions:**
- Use a large prime modulus (~10^9+7 or 10^9+9).
- Use a base larger than the alphabet size.
- For adversarial inputs, use **double hashing** (two independent (p, m) pairs) — a collision then needs both to coincide.

---

## Code (C++)
```cpp
// Precompute prefix hashes and powers, then answer substring-hash queries in O(1).
struct StringHash {
    const long long p = 31, m = 1e9 + 9;
    vector<long long> h;       // h[i] = hash of s[0..i-1]
    vector<long long> pw;      // pw[i] = p^i mod m

    StringHash(const string& s) {
        int n = s.size();
        h.assign(n + 1, 0);
        pw.assign(n + 1, 1);
        for (int i = 0; i < n; i++) {
            // (s[i]-'a'+1) keeps 'a' from hashing to 0 (so "a","aa" differ)
            h[i + 1] = (h[i] + (s[i] - 'a' + 1) * pw[i]) % m;
            pw[i + 1] = (pw[i] * p) % m;
        }
    }

    // Hash of s[l..r], scaled by p^l (compare only equal-length substrings,
    // OR multiply the shorter one's hash by the power difference to align).
    long long sub(int l, int r) {
        return ((h[r + 1] - h[l]) % m + m) % m;   // +m then %m fixes negatives
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N) preprocessing, O(1) per substring-hash query |
| Space | O(N) — prefix-hash and power arrays |

> ⚠️ Always take `(x % m + m) % m` after a subtraction: modular arithmetic on signed integers can go negative. Multiplications must use `long long` to avoid overflow before the `% m`.
