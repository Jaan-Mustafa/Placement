# Longest Word With All Prefixes  🟡

**Reference:** [LeetCode 720 — Longest Word in Dictionary](https://leetcode.com/problems/longest-word-in-dictionary/) · [GeeksforGeeks — Longest word with all prefixes (Complete String)](https://www.geeksforgeeks.org/find-the-longest-string-that-can-be-made-up-of-other-strings-from-an-array/)

---

## Problem
Given a list of words, find the **longest word** such that **every prefix** of it (of length 1, 2, ... up to its length) is also present in the list. If multiple qualify, return the **lexicographically smallest**. If none, return `""`.

```
Input:  ["n","ni","nin","ninj","ninja","ninj"]
Output: "ninja"          // n, ni, nin, ninj, ninja all present

Input:  ["ab","bc","cd"]
Output: ""               // "ab" needs "a" too; none qualify
```

---

## Intuition
Insert every word into a trie (with `isEnd`). A word is valid iff, walking its path from the root, **every node along the way is marked `isEnd`** — that guarantees each prefix is itself an inserted word. Among all valid words pick the longest; break ties by lexicographic order. A DFS from the root that only descends through `isEnd` nodes naturally enumerates valid words; visiting children in `a..z` order gives the lexicographically smallest answer for free.

---

## Code (C++)
```cpp
class Solution {
    struct Node {
        Node* children[26] = {nullptr};
        bool isEnd = false;
    };
    Node* root;

    void insert(const string& w) {
        Node* cur = root;
        for (char c : w) {
            int idx = c - 'a';
            if (!cur->children[idx]) cur->children[idx] = new Node();
            cur = cur->children[idx];
        }
        cur->isEnd = true;
    }

    string best;
    void dfs(Node* node, string& path) {
        // path is a valid word (every prefix is an inserted word).
        // Longer wins; equal length -> lexicographically smaller wins.
        if (path.size() > best.size()) best = path;
        for (int i = 0; i < 26; i++) {                 // a..z order
            Node* nxt = node->children[i];
            if (nxt && nxt->isEnd) {                   // only valid prefixes
                path.push_back('a' + i);
                dfs(nxt, path);
                path.pop_back();
            }
        }
    }

public:
    string longestWord(vector<string>& words) {
        root = new Node();
        for (auto& w : words) insert(w);
        best = "";
        string path;
        dfs(root, path);          // root's empty prefix is implicitly valid
        return best;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(total chars) to build + O(total chars) to DFS |
| Space | O(total chars × 26) for the trie |

> ⚠️ Tie-breaking: because we scan children `a → z` and only replace `best` on a **strictly longer** path, the first word of any given length we reach is the lexicographically smallest — so `>` (not `>=`) is essential.
