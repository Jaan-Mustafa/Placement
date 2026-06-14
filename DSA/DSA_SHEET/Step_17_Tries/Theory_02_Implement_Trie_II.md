# Implement Trie II (count words / prefixes)  🟡

**Reference:** [LeetCode 1804 — Implement Trie II (Prefix Tree)](https://leetcode.com/problems/implement-trie-ii-prefix-tree/) · [GeeksforGeeks — Trie with word/prefix counts](https://www.geeksforgeeks.org/trie-insert-and-search/)

---

## Problem
A richer trie that also supports deletion and exact counts:
- `insert(word)` — add a word (duplicates allowed).
- `countWordsEqualTo(word)` — how many times `word` was inserted.
- `countWordsStartingWith(prefix)` — how many inserted words have this prefix.
- `erase(word)` — remove one occurrence of `word`.

```
insert("apple"); insert("apple"); insert("app")
countWordsEqualTo("apple")      -> 2
countWordsStartingWith("app")   -> 3
erase("apple")
countWordsEqualTo("apple")      -> 1
```

---

## Intuition
Plain `isEnd` can't count. Replace it with two integer counters per node:
- `cntEnd` — number of words that **end** at this node.
- `cntPrefix` — number of words that **pass through** this node (i.e. how many words have the path-so-far as a prefix).

Insert increments `cntPrefix` on every node visited and `cntEnd` at the last one. Erase decrements them symmetrically. The two queries are then just lookups of these counters.

---

## Code (C++)
```cpp
class Trie {
    struct Node {
        Node* children[26] = {nullptr};
        int cntEnd = 0;     // words ending exactly here
        int cntPrefix = 0;  // words passing through here
    };
    Node* root;

    Node* walk(const string& s) {            // node at end of path, or null
        Node* cur = root;
        for (char c : s) {
            int idx = c - 'a';
            if (!cur->children[idx]) return nullptr;
            cur = cur->children[idx];
        }
        return cur;
    }

public:
    Trie() { root = new Node(); }

    void insert(string word) {
        Node* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx])
                cur->children[idx] = new Node();
            cur = cur->children[idx];
            cur->cntPrefix++;                 // one more word through here
        }
        cur->cntEnd++;                        // one more word ends here
    }

    int countWordsEqualTo(string word) {
        Node* node = walk(word);
        return node ? node->cntEnd : 0;
    }

    int countWordsStartingWith(string prefix) {
        Node* node = walk(prefix);
        return node ? node->cntPrefix : 0;
    }

    void erase(string word) {
        // caller guarantees word exists; mirror the insert decrements
        Node* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            cur = cur->children[idx];
            cur->cntPrefix--;
        }
        cur->cntEnd--;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(L) per operation |
| Space | O(total chars × 26) |

> ⚠️ `erase` assumes the word was previously inserted (per the problem constraints). If you can't assume that, walk first to verify the path exists and `cntEnd > 0` before decrementing.
