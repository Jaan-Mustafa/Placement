# LFU Cache  🔴

**Reference:** [LeetCode 460 — LFU Cache](https://leetcode.com/problems/lfu-cache/)

---

## Problem
Design a cache with capacity `c` and O(1) `get`/`put` that evicts the **least frequently used** key. On a frequency tie, evict the least recently used among them.

```
put(1,1), put(2,2), get(1)->1 (freq1=2), put(3,3) evicts 2 (freq=1), get(2)->-1
```

---

## Intuition
Bucket keys by frequency: `freqList[f]` is a doubly linked list of keys with frequency `f`, ordered by recency (front = most recent). A map gives each key its value, frequency, and list iterator. Track `minFreq`. On access, move the key from bucket `f` to `f+1`, updating `minFreq` when a bucket empties. Eviction removes the LRU key from the `minFreq` bucket — all O(1).

---

## Code (C++)
```cpp
class LFUCache {
    int cap, minFreq;
    unordered_map<int,int> valueOf, freqOf;                  // key -> value / freq
    unordered_map<int, list<int>> freqList;                  // freq -> keys (front=recent)
    unordered_map<int, list<int>::iterator> iterOf;          // key -> position
public:
    LFUCache(int capacity) : cap(capacity), minFreq(0) {}

    void touch(int key) {                                    // bump frequency
        int f = freqOf[key];
        freqList[f].erase(iterOf[key]);
        if (freqList[f].empty()) {
            freqList.erase(f);
            if (minFreq == f) minFreq++;                     // bucket emptied
        }
        freqOf[key] = f + 1;
        freqList[f + 1].push_front(key);
        iterOf[key] = freqList[f + 1].begin();
    }
    int get(int key) {
        if (!valueOf.count(key)) return -1;
        touch(key);
        return valueOf[key];
    }
    void put(int key, int value) {
        if (cap == 0) return;
        if (valueOf.count(key)) {                            // update + bump
            valueOf[key] = value;
            touch(key);
            return;
        }
        if ((int)valueOf.size() == cap) {                    // evict LFU+LRU
            int evict = freqList[minFreq].back();
            freqList[minFreq].pop_back();
            if (freqList[minFreq].empty()) freqList.erase(minFreq);
            valueOf.erase(evict); freqOf.erase(evict); iterOf.erase(evict);
        }
        valueOf[key] = value;
        freqOf[key] = 1;
        freqList[1].push_front(key);
        iterOf[key] = freqList[1].begin();
        minFreq = 1;                                         // new key has freq 1
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(1) for both get and put |
| Space | O(capacity) |

> ⚠️ `minFreq` resets to 1 on every insertion (new keys start at frequency 1) and only increments inside `touch` when the current min bucket empties. The tail of a frequency bucket is the LRU tie-breaker.
