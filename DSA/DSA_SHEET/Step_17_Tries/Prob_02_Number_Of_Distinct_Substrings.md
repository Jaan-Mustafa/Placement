# Number of Distinct Substrings  🔴

**Reference:** [GeeksforGeeks — Count distinct substrings of a string using Trie](https://www.geeksforgeeks.org/count-number-of-distinct-substring-in-a-string/) · [Coding Ninjas / Naukri — Count distinct substrings](https://www.naukri.com/code360/problems/count-distinct-substrings_985292)

---

## Problem
Given a string `s`, count the number of **distinct non-empty substrings** of `s`. (Some variants include the empty substring; we count non-empty here.)

```
Input:  "ababa"
Output: 9
Distinct substrings: {a, b, ab, ba, aba, bab, abab, baba, ababa}  -> 9
```

---

## Intuition
Every substring of `s` is a **prefix of some suffix** of `s`. So insert all suffixes into a trie. While inserting, each time we **create a new node**, we have discovered a new distinct substring (the path to that node). Counting created nodes (excluding the root) therefore counts distinct substrings exactly — duplicate substrings reuse existing nodes and add nothing.

---

## Code (C++)
```cpp
struct Node {
    Node* children[26] = {nullptr};
};

int countDistinctSubstrings(string s) {
    Node* root = new Node();
    int count = 0;                              // distinct substrings = new nodes
    int n = s.size();

    for (int i = 0; i < n; i++) {               // insert suffix starting at i
        Node* cur = root;
        for (int j = i; j < n; j++) {
            int idx = s[j] - 'a';
            if (!cur->children[idx]) {
                cur->children[idx] = new Node(); // new node = new distinct substring
                count++;
            }
            cur = cur->children[idx];
        }
    }
    return count;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) — insert all N suffixes, each up to length N |
| Space | O(N²) nodes in the worst case (all-distinct chars) |

> ⚠️ The trie approach is O(N²) in both time and space. For very large strings, a **suffix automaton** or **suffix array + LCP** counts distinct substrings in O(N) / O(N log N) — but the trie version is the standard interview answer and is easiest to reason about.
