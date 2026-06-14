# map & multimap — Ordered Key→Value

`#include <map>`

A balanced BST of key→value pairs, **sorted by key**. `map` has unique keys; `multimap` allows duplicate keys. All ops O(log n), with ordered queries (`lower_bound`, iterate in sorted key order) you can't get from a hash map.

---

## Construction
```cpp
map<string, int> mp;
map<int, string> m2 = {{1,"a"}, {2,"b"}};
map<int, int, greater<int>> desc;            // keys in descending order
multimap<int, int> mm;                        // duplicate keys allowed
```

---

## Full function list

### Access
| Function | Description | Complexity |
|---|---|---|
| `mp[key]` | Access; **inserts default if absent** | O(log n) |
| `mp.at(key)` | Access; **throws** if absent (no insert) | O(log n) |

### Modify
| Function | Description | Complexity |
|---|---|---|
| `mp.insert({k, v})` | Insert; **no overwrite** if k exists | O(log n) |
| `mp.insert_or_assign(k, v)` | Insert or overwrite (C++17) | O(log n) |
| `mp.emplace(k, v)` | Construct + insert | O(log n) |
| `mp.try_emplace(k, v)` | Insert only if absent (C++17) | O(log n) |
| `mp.erase(key)` | Erase by key (returns # removed) | O(log n) |
| `mp.erase(it)` | Erase at iterator (returns next it) | O(log n) |
| `mp.clear()` | Remove all | O(n) |

### Lookup
| Function | Description | Returns |
|---|---|---|
| `mp.find(key)` | Locate key | iterator or `mp.end()` |
| `mp.count(key)` | 0/1 (map); #dups (multimap) | size_t |
| `mp.contains(key)` | C++20 | bool |
| `mp.lower_bound(k)` | First entry with key **≥ k** | iterator |
| `mp.upper_bound(k)` | First entry with key **> k** | iterator |
| `mp.equal_range(k)` | `{lower, upper}` | pair |

### Iterate
| Function | Description |
|---|---|
| `mp.begin()/end()` | iterate in **sorted key** order |
| `mp.rbegin()/rend()` | reverse (largest key first) |
| `mp.size()/empty()` | size / emptiness |

Each element is a `pair<const Key, Value>` — `it->first` is the key, `it->second` the value.

---

## Examples

### Frequency counting (most common use)
```cpp
map<string,int> freq;
freq["apple"]++;          // auto-creates "apple"=0, then ++ -> 1
freq["apple"]++;          // 2
freq["banana"] = 5;

// iterate in sorted key order
for (auto& [key, val] : freq)              // structured binding (C++17)
    cout << key << " -> " << val << "\n";  // apple -> 2 / banana -> 5

// pre-C++17
for (auto it = freq.begin(); it != freq.end(); ++it)
    cout << it->first << " " << it->second;
```

### ⚠️ The `[]` auto-insert trap
```cpp
map<int,int> mp;
if (mp[5] == 0) { /* ... */ }   // BUG: this CREATED key 5 with value 0!
cout << mp.size();              // 1  (oops)

// to CHECK without inserting, use find / count / contains:
if (mp.find(5) != mp.end()) { /* exists */ }
if (mp.count(5)) { /* exists */ }
if (mp.contains(5)) { /* C++20 */ }
```

### insert vs [] vs insert_or_assign
```cpp
map<int,string> m;
m[1] = "a";
m.insert({1, "b"});             // NO effect — key 1 exists, value stays "a"
m[1] = "b";                     // overwrites -> "b"
m.insert_or_assign(1, "c");     // overwrites -> "c"
auto [it, inserted] = m.insert({2, "x"});   // inserted == true
```

### Ordered queries
```cpp
map<int,string> m = {{10,"a"},{20,"b"},{30,"c"}};

auto it = m.lower_bound(15);    // -> {20,"b"}  (first key >= 15)
auto it2 = m.upper_bound(20);   // -> {30,"c"}  (first key > 20)

cout << m.begin()->first;       // 10  (smallest key)
cout << m.rbegin()->first;      // 30  (largest key)
```

### Erase while iterating
```cpp
for (auto it = mp.begin(); it != mp.end(); ) {
    if (it->second == 0) it = mp.erase(it);   // erase returns next iterator
    else ++it;
}
```

### multimap — duplicate keys
```cpp
multimap<int,string> mm;
mm.insert({1, "a"});
mm.insert({1, "b"});            // both kept
cout << mm.count(1);            // 2

auto [lo, hi] = mm.equal_range(1);
for (auto it = lo; it != hi; ++it) cout << it->second;   // a b
// multimap has NO operator[] (ambiguous with dup keys)
```

---

## DSA patterns
- **Frequency map with sorted keys:** histogram problems where you process keys in order.
- **TreeMap-style queries:** floor/ceil of a key, "next event ≥ time" → `lower_bound`/`upper_bound`. Calendar booking, interval scheduling.
- **Ordered counter for sweep-line:** insert/remove events keyed by coordinate.
- **`map<int,int>` as a sorted multiset with counts** (increment/decrement counts instead of multiset insert/erase).

## map vs unordered_map
| | map | unordered_map |
|---|---|---|
| Order | sorted by key | none |
| find/insert/erase | O(log n) | O(1) avg |
| `lower_bound`/range/min/max key | ✅ | ❌ |
| Worst case | O(log n) | O(n) |

Use `unordered_map` for plain key→value lookups (faster); use `map` only when you need sorted keys or range queries.

## Gotchas
- ⚠️ `mp[key]` **creates** the key if missing — never use it for read-only checks. Use `find`/`count`/`contains`.
- ⚠️ Keys are **const** — you can change `it->second` but not `it->first`.
- ⚠️ `multimap` has no `operator[]`. Use `insert` + `equal_range`.
- ⚠️ Use the **member** `lower_bound`, not the `<algorithm>` free function (O(n) on a map).
