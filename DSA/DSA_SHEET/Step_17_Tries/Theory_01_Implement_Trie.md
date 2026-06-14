# Implement Trie (insert, search, startsWith)  🟡

**Reference:** [LeetCode 208 — Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/)

---

## Problem
Build a prefix tree supporting:
- `insert(word)` — add a word.
- `search(word)` — return true if the **exact** word was inserted.
- `startsWith(prefix)` — return true if any inserted word has this prefix.

```
insert("apple")
search("apple")   -> true
search("app")     -> false
startsWith("app") -> true
insert("app")
search("app")     -> true
```

---

## Intuition
A trie stores strings as paths from the root, one edge per character. Each node holds 26 child pointers (one per lowercase letter) and a flag `isEnd` marking that a word terminates here. Insert walks/creates the path; `search` walks and checks `isEnd` at the end; `startsWith` walks and only checks that the path exists.

---

## Code (C++)
```cpp
class Trie {
    struct Node {
        Node* children[26] = {nullptr};  // child per letter 'a'..'z'
        bool isEnd = false;              // true if a word ends here
    };
    Node* root;

    // walk the trie following `s`; return the node at the end, or null
    Node* walk(const string& s) {
        Node* cur = root;
        for (char c : s) {
            int idx = c - 'a';
            if (!cur->children[idx]) return nullptr;  // path broken
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
            if (!cur->children[idx])           // create node if missing
                cur->children[idx] = new Node();
            cur = cur->children[idx];
        }
        cur->isEnd = true;                     // mark end of word
    }

    bool search(string word) {
        Node* node = walk(word);
        return node && node->isEnd;            // must be an exact word end
    }

    bool startsWith(string prefix) {
        return walk(prefix) != nullptr;        // path existing is enough
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(L) per op, where L = length of the word/prefix |
| Space | O(total chars × 26) across all inserted words |

> ⚠️ `search` must check `isEnd`, not just path existence — otherwise `"app"` would match after only `"apple"` was inserted. `startsWith` deliberately skips that check.
