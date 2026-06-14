# set & multiset — Ordered, Balanced BST

`#include <set>`

A self-balancing binary search tree (typically red-black). Keeps elements **sorted** at all times. `set` stores **unique** keys; `multiset` allows **duplicates**. Everything is O(log n), and you get ordered queries (`lower_bound`, predecessor/successor) that hash containers can't do.

---

## Construction
```cpp
set<int> s;                                 // ascending (default less<int>)
set<int, greater<int>> s2;                  // descending order
set<int> s3 = {5, 1, 3, 1};                 // stored as {1,3,5} (sorted, unique)
multiset<int> ms = {1, 1, 2, 3};            // keeps {1,1,2,3}
set<int> s4(v.begin(), v.end());            // from a vector (dedupe + sort)
```

---

## Full function list

### Modify
| Function | Description | Complexity |
|---|---|---|
| `s.insert(x)` | Insert; in `set`, ignored if exists | O(log n) |
| `s.emplace(args)` | Construct + insert | O(log n) |
| `s.erase(x)` | Erase **ALL** elements equal to x | O(log n) |
| `s.erase(it)` | Erase the single element at iterator | O(log n) (amortized O(1)) |
| `s.erase(first, last)` | Erase a range | O(log n + k) |
| `s.clear()` | Remove all | O(n) |
| `s.merge(s2)` | Splice all nodes from s2 (C++17) | O(n log n) |
| `s.extract(it)` | Remove & return a node handle (C++17) | O(log n) |

### Lookup
| Function | Description | Returns |
|---|---|---|
| `s.find(x)` | Locate x | iterator or `s.end()` |
| `s.count(x)` | set: 0/1; multiset: #occurrences | size_t |
| `s.contains(x)` | C++20 | bool |
| `s.lower_bound(x)` | First element **≥ x** | iterator |
| `s.upper_bound(x)` | First element **> x** | iterator |
| `s.equal_range(x)` | `{lower_bound, upper_bound}` pair | pair of iterators |

### Access / capacity / iterators
| Function | Description |
|---|---|
| `*s.begin()` | smallest element (O(1)) |
| `*s.rbegin()` | largest element (O(1)) |
| `s.size()` / `s.empty()` | size / emptiness |
| `begin/end/rbegin/rend` | bidirectional iterators (sorted order) |

> ⚠️ No random access — `s[i]` does NOT exist. No `s.front()`/`s.back()` either; use `*begin()` / `*rbegin()`.

---

## Examples

### set basics
```cpp
set<int> s = {5, 1, 3, 1};         // {1,3,5}
s.insert(4);                       // {1,3,4,5}
s.insert(3);                       // no-op (already present)
s.erase(3);                        // {1,4,5}

cout << s.count(4);                // 1
cout << (s.find(7) != s.end());    // 0 (not found)

cout << *s.begin();                // 1   (min)
cout << *s.rbegin();               // 5   (max)
cout << *prev(s.end());            // 5   (also max)

for (int x : s) cout << x << " ";  // 1 4 5  (always sorted)
```

### lower_bound / upper_bound (ordered queries)
```cpp
set<int> s = {10, 20, 30, 40};

auto lb = s.lower_bound(20);   // -> 20  (first >= 20)
auto ub = s.upper_bound(20);   // -> 30  (first > 20)

// largest element <= x  (predecessor / floor)
auto it = s.upper_bound(25);   // -> 30
if (it != s.begin()) cout << *prev(it);   // 20

// smallest element >= x  (ceil)
auto it2 = s.lower_bound(25);  // -> 30
if (it2 != s.end()) cout << *it2;          // 30
```
> ⚠️ Use the **member** `s.lower_bound(x)` — O(log n). The free function `lower_bound(s.begin(), s.end(), x)` on a set is **O(n)** because set iterators aren't random-access. Same warning for `find`/`count`.

### multiset — duplicates
```cpp
multiset<int> ms = {1, 1, 2, 2, 2, 3};
cout << ms.count(2);               // 3
ms.insert(2);                      // now four 2s

// ⚠️ erase(value) removes ALL copies
ms.erase(2);                       // removes all four 2s!

// erase ONE copy -> erase via iterator
multiset<int> m = {5, 5, 5};
m.erase(m.find(5));                // removes a single 5  -> {5,5}
cout << m.size();                  // 2
```

### equal_range
```cpp
multiset<int> ms = {1, 2, 2, 2, 3};
auto [lo, hi] = ms.equal_range(2);
for (auto it = lo; it != hi; ++it) cout << *it;   // 2 2 2
cout << distance(lo, hi);          // 3 (count of 2s)
```

---

## DSA patterns

### 1. Running median / order maintenance
Keep a sorted multiset; insert each value; the median is the middle iterator (advance with `next(ms.begin(), ms.size()/2)`).

### 2. Sliding window with removable max/min
```cpp
multiset<int> window;
window.insert(a[i]);                     // add new
window.erase(window.find(a[i - k]));     // remove the one leaving (find!)
int mx = *window.rbegin();               // current window max
int mn = *window.begin();                // current window min
```

### 3. Floor / ceil queries
`lower_bound`/`upper_bound` give "smallest ≥ x" and "largest ≤ x" — used in interval scheduling, nearest-value problems, calendar booking.

### 4. Coordinate compression
```cpp
set<int> uniq(all.begin(), all.end());   // sorted unique values
// map each value to its rank for compressed indexing
```

---

## set vs unordered_set
| | set | unordered_set |
|---|---|---|
| Order | sorted | none |
| find/insert/erase | O(log n) | O(1) avg |
| `lower_bound`/range/min/max | ✅ | ❌ |
| Worst case | O(log n) | O(n) (bad hash) |

Use `set` only when you need **ordering**; otherwise `unordered_set` is faster.

## Gotchas
- ⚠️ `multiset.erase(value)` deletes **every** matching element. Use `erase(find(value))` to remove just one.
- ⚠️ Use **member** `lower_bound`/`find` on sets, not the `<algorithm>` free functions (those are O(n) here).
- ⚠️ Elements are **immutable** through iterators (changing a key would break the tree order). To "modify", erase + reinsert.
- ⚠️ No `[]`, no `front()`/`back()`. Min = `*begin()`, Max = `*rbegin()`.
