# unordered_set — Hash Set

`#include <unordered_set>`

A hash table of unique keys. **Average O(1)** insert/find/erase, but **no ordering** and O(n) worst case under hash collisions. Reach for it whenever you only need "have I seen this?" and don't care about sorted order.

---

## Construction
```cpp
unordered_set<int> us;
unordered_set<int> us2 = {1, 2, 3, 2};      // stores {1,2,3} (order arbitrary)
unordered_set<string> words;
unordered_set<int> us3(v.begin(), v.end()); // from a vector
```

---

## Full function list

### Modify / lookup
| Function | Description | Avg | Worst |
|---|---|---|---|
| `us.insert(x)` | Insert (no-op if present) | O(1) | O(n) |
| `us.emplace(args)` | Construct + insert | O(1) | O(n) |
| `us.erase(x)` | Erase by value | O(1) | O(n) |
| `us.erase(it)` | Erase at iterator | O(1) | O(n) |
| `us.find(x)` | Iterator or `us.end()` | O(1) | O(n) |
| `us.count(x)` | 0 or 1 | O(1) | O(n) |
| `us.contains(x)` | C++20, bool | O(1) | O(n) |
| `us.clear()` | Remove all | O(n) | — |

### Capacity / hashing controls
| Function | Description |
|---|---|
| `us.size()` / `us.empty()` | size / emptiness |
| `us.reserve(n)` | Pre-size buckets for n elements (avoids rehashing) |
| `us.load_factor()` | size / bucket_count |
| `us.max_load_factor(x)` | Set threshold that triggers rehash |
| `us.bucket_count()` | Number of buckets |
| `us.rehash(n)` | Set bucket count to at least n |

> No `lower_bound`, no `rbegin`, no sorted iteration — that's the trade for O(1).

---

## Examples

### Basics
```cpp
unordered_set<int> us;
us.insert(5);
us.insert(5);                      // no-op
us.insert(10);
cout << us.size();                 // 2

cout << us.count(5);               // 1
cout << (us.find(99) != us.end()); // 0
us.erase(5);                       // {10}

if (us.contains(10)) cout << "yes";   // C++20
```

### "Seen before?" — duplicate detection
```cpp
bool hasDuplicate(vector<int>& v) {
    unordered_set<int> seen;
    for (int x : v) {
        if (seen.count(x)) return true;   // already saw it
        seen.insert(x);
    }
    return false;
}
```

### Longest consecutive sequence (classic O(n) use)
```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> s(nums.begin(), nums.end());
    int best = 0;
    for (int x : s) {
        if (!s.count(x - 1)) {            // x is a sequence start
            int cur = x, len = 1;
            while (s.count(cur + 1)) { cur++; len++; }
            best = max(best, len);
        }
    }
    return best;
}
```

### Two-sum style membership
```cpp
// does any pair sum to target?
unordered_set<int> seen;
for (int x : nums) {
    if (seen.count(target - x)) return true;
    seen.insert(x);
}
```

---

## Hashing custom types

`unordered_set` needs a hash for its key type. Built-in for `int`, `string`, etc. For `pair`/custom structs you must supply one:

```cpp
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<long long>()((long long)p.first * 1000000007 + p.second);
    }
};
unordered_set<pair<int,int>, PairHash> visited;   // grid cells visited
visited.insert({r, c});
```

---

## Performance: avoid anti-hash TLE
- ⚠️ On adversarial inputs (Codeforces hacks), the default hash can be forced into O(n) per op → TLE. Mitigations:
  - `us.reserve(n)` upfront to cut rehashing.
  - For `unordered_map<int,int>`, attach a randomized custom hash, or just use `map`/`set` (O(log n) guaranteed) when n is small.
- For small key ranges (e.g., 0..1e5), a plain `vector<bool>`/`bool[]` is faster than any hash set.

## unordered_set vs set
| | unordered_set | set |
|---|---|---|
| Speed | O(1) avg | O(log n) |
| Order / range queries | ❌ | ✅ |
| Worst case | O(n) | O(log n) |
| Memory | higher (buckets) | lower |

Default to `unordered_set` for plain membership; switch to `set` if you need sorted order, `lower_bound`, or guaranteed worst case.

## When to use
- Membership / "seen" checks, dedup.
- Two-sum, subarray-sum-exists, cycle detection on values.
- Visited-set for graph/grid search when you can't index directly.

## Gotchas
- ⚠️ No ordering — iterating gives arbitrary (and version-dependent) order.
- ⚠️ Needs a custom hash for `pair`/`tuple`/structs.
- ⚠️ Iterators invalidated by rehash (on insert that grows buckets); references to elements stay valid.
