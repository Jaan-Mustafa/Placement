# unordered_map — Hash Map

`#include <unordered_map>`

A hash table of key→value pairs. **Average O(1)** access, no key ordering, O(n) worst case. The single most-used container for frequency counts, memoization, and lookup tables in DSA.

---

## Construction
```cpp
unordered_map<int,int> mp;
unordered_map<string,int> freq;
unordered_map<int,vector<int>> groups;
unordered_map<char,int> m = {{'a',1},{'b',2}};
```

---

## Full function list

### Access
| Function | Description | Avg | Worst |
|---|---|---|---|
| `mp[key]` | Access; **inserts default if absent** | O(1) | O(n) |
| `mp.at(key)` | Access; **throws** if absent | O(1) | O(n) |

### Modify / lookup
| Function | Description | Avg |
|---|---|---|
| `mp.insert({k,v})` | Insert (no overwrite) | O(1) |
| `mp.insert_or_assign(k,v)` | Insert or overwrite (C++17) | O(1) |
| `mp.emplace(k,v)` / `try_emplace(k,v)` | Construct + insert | O(1) |
| `mp.erase(key)` | Erase by key | O(1) |
| `mp.find(key)` | Iterator or `end()` | O(1) |
| `mp.count(key)` | 0/1 | O(1) |
| `mp.contains(key)` | C++20 bool | O(1) |
| `mp.clear()` | Remove all | O(n) |

### Hash control
| Function | Description |
|---|---|
| `mp.reserve(n)` | Pre-size buckets (cuts rehashing → faster, safer) |
| `mp.load_factor()` / `max_load_factor(x)` | Tune rehash threshold |
| `mp.bucket_count()` / `rehash(n)` | Bucket internals |

> No `lower_bound`, no sorted iteration — that's the trade for O(1).

---

## Examples

### Frequency counting
```cpp
unordered_map<int,int> freq;
vector<int> a = {1, 2, 2, 3, 3, 3};
for (int x : a) freq[x]++;          // {1:1, 2:2, 3:3}

cout << freq[3];                    // 3
cout << freq.count(2);              // 1

// find the most frequent element
int best = -1, bestCnt = 0;
for (auto& [val, cnt] : freq)
    if (cnt > bestCnt) { bestCnt = cnt; best = val; }
```

### Two Sum (the canonical O(n) use)
```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> seen;        // value -> index
    for (int i = 0; i < nums.size(); i++) {
        int need = target - nums[i];
        if (seen.count(need))
            return {seen[need], i};
        seen[nums[i]] = i;
    }
    return {};
}
```

### Memoization (top-down DP)
```cpp
unordered_map<int,long long> memo;
long long fib(int n) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];      // cache hit
    return memo[n] = fib(n-1) + fib(n-2);   // store & return
}
```

### Group anagrams (sorted key → bucket)
```cpp
unordered_map<string, vector<string>> groups;
for (string& w : words) {
    string key = w;
    sort(key.begin(), key.end());           // anagrams share sorted form
    groups[key].push_back(w);
}
```

### Subarray sum equals K (prefix sum + hashmap)
```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int,int> cnt{{0,1}};      // prefix sum -> frequency
    int sum = 0, res = 0;
    for (int x : nums) {
        sum += x;
        res += cnt[sum - k];                // how many prefixes give a k-subarray
        cnt[sum]++;
    }
    return res;
}
```

---

## Custom-type keys need a hash

```cpp
// pair as a key
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<long long>()(((long long)p.first << 32) ^ p.second);
    }
};
unordered_map<pair<int,int>, int, PairHash> grid;
grid[{2,3}] = 5;
```

For built-in keys (`int`, `long long`, `string`, `char`) no hash is needed.

---

## Anti-hash / TLE safety (competitive)
```cpp
// custom 64-bit hash to defeat adversarial test cases (Codeforces hacks)
struct FastHash {
    static uint64_t splitmix64(uint64_t x) {
        x += 0x9e3779b97f4a7c15;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
        x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
        return x ^ (x >> 31);
    }
    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED = 0x123456789abcdef;   // pass a per-run seed via args
        return splitmix64(x + FIXED);
    }
};
unordered_map<long long, int, FastHash> safe;
```
- ⚠️ Also call `mp.reserve(n)` and optionally `mp.max_load_factor(0.25)` for speed on large n.
- If n is small or keys are dense small ints, a plain array `int cnt[N]` beats any hash map.

## unordered_map vs map
| | unordered_map | map |
|---|---|---|
| Speed | O(1) avg | O(log n) |
| Order / range queries | ❌ | ✅ sorted, lower_bound |
| Worst case | O(n) | O(log n) |
| Memory | higher | lower |

Default to `unordered_map`; switch to `map` when you need sorted keys, range queries, or guaranteed worst case.

## When to use
- Frequency tables, counting, dedup-with-data.
- Memoization caches.
- Index/lookup tables (value → position, name → record).
- Prefix-sum / complement lookups (two-sum, subarray sum).

## Gotchas
- ⚠️ `mp[key]` **auto-creates** the key — use `find`/`count`/`contains` for read-only checks.
- ⚠️ `[]` requires the value type be default-constructible; `at()` doesn't insert.
- ⚠️ Needs a custom hash for `pair`/`tuple`/structs.
- ⚠️ Iteration order is arbitrary and unstable across runs; rehash invalidates iterators.
