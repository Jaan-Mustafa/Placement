# LRU Cache  🔴

**Reference:** [LeetCode 146 — LRU Cache](https://leetcode.com/problems/lru-cache/)

---

## Problem
Design a cache with capacity `c` supporting `get(key)` and `put(key, value)` in O(1). When full, evict the **least recently used** entry.

```
put(1,1), put(2,2), get(1)->1, put(3,3) evicts 2, get(2)->-1
```

---

## Intuition
Combine a **doubly linked list** (ordering by recency: most-recent at the front, least-recent at the tail) with a **hash map** from key to its list node. `get`/`put` move the touched node to the front in O(1); eviction removes the tail. C++ `std::list` plus `unordered_map<int, list::iterator>` makes this clean with `splice` for the move.

---

## Code (C++)
```cpp
class LRUCache {
    int cap;
    list<pair<int,int>> dll;                         // front = most recent
    unordered_map<int, list<pair<int,int>>::iterator> mp;
public:
    LRUCache(int capacity) : cap(capacity) {}

    int get(int key) {
        auto it = mp.find(key);
        if (it == mp.end()) return -1;
        dll.splice(dll.begin(), dll, it->second);    // move node to front, O(1)
        return it->second->second;
    }
    void put(int key, int value) {
        auto it = mp.find(key);
        if (it != mp.end()) {                        // update existing
            it->second->second = value;
            dll.splice(dll.begin(), dll, it->second);
            return;
        }
        if ((int)dll.size() == cap) {                // evict LRU (tail)
            mp.erase(dll.back().first);
            dll.pop_back();
        }
        dll.push_front({key, value});
        mp[key] = dll.begin();
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(1) for both get and put |
| Space | O(capacity) |

> `splice` re-links the existing node without invalidating its iterator, so the map entry stays valid — that is what keeps the move O(1).
