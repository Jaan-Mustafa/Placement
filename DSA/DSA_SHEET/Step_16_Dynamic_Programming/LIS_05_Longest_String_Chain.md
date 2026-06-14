# Longest String Chain  🟡

**Reference:** https://leetcode.com/problems/longest-string-chain/

---

## Problem

Given an array of `words`, a word `A` is a **predecessor** of word `B` if inserting exactly one character anywhere in `A` (without reordering) makes it equal to `B`. A word chain is a sequence where each word is a predecessor of the next. Return the length of the longest possible word chain.

```
Input:  words = ["a","b","ba","bca","bda","bdca"]
Output: 4
Explanation: "a" -> "ba" -> "bda" -> "bdca" (each adds one char).
```

---

## Intuition

This is LIS where "comparison" is the predecessor relation. **Sort words by length** so any predecessor of a word appears before it.

Let `dp[i]` = longest chain ending at `words[i]`. For each `j < i`, if `words[j]` is a predecessor of `words[i]` (its length is exactly one less and it is obtainable by deleting one char from `words[i]`), then `words[i]` can extend that chain.

```
dp[i] = 1 + max over j<i (dp[j])  if words[j] is predecessor of words[i]
```

The `isPredecessor` check tries deleting each character of the longer word once and compares against the shorter word.

---

## Code (C++)

```cpp
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

// true if 'a' becomes 'b' by inserting exactly one char (|b| == |a| + 1)
bool isPredecessor(const string& a, const string& b) {
    if (b.size() != a.size() + 1) return false;
    int i = 0, j = 0, skips = 0;
    while (i < (int)a.size() && j < (int)b.size()) {
        if (a[i] == b[j]) { i++; j++; }
        else { j++; skips++; }     // skip one char of the longer word
        if (skips > 1) return false;
    }
    return true;                   // remaining trailing char (if any) is the inserted one
}

int longestStrChain(vector<string>& words) {
    int n = words.size();
    // sort by length so predecessors come first
    sort(words.begin(), words.end(),
         [](const string& x, const string& y){ return x.size() < y.size(); });

    vector<int> dp(n, 1);
    int best = 1;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (isPredecessor(words[j], words[i]) && dp[j] + 1 > dp[i]) {
                dp[i] = dp[j] + 1;
            }
        }
        best = max(best, dp[i]);
    }
    return best;
}
```

Space is O(n) for `dp`. A common speed-up uses a hash map `dp[word]`: for each word, delete each character to form candidate predecessors and look them up in O(L) per deletion, giving O(N * L^2).

---

## Complexity

| | |
|---|---|
| Time | O(n^2 * L)  (L = max word length) |
| Space | O(n) |

> ⚠️ Sorting by length is essential — without it a predecessor might appear after its successor and be missed. The predecessor check requires the length to differ by **exactly one**; equal-length words are never in a chain together.
