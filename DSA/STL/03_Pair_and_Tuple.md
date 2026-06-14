# pair & tuple — Grouping Values

`#include <utility>` (pair) · `#include <tuple>` (tuple)

Lightweight fixed-size bundles. `pair` holds exactly 2 values, `tuple` holds any fixed number. Both are comparable (lexicographically), so they sort and work as map/set keys.

---

## pair

### Construction & access
```cpp
pair<int, string> p = {1, "abc"};
pair<int, string> p2 = make_pair(2, "xyz");
pair<int, string> p3(3, "pqr");

cout << p.first;     // 1
cout << p.second;    // "abc"
p.first = 10;        // mutable
```

### Comparison (lexicographic: first, then second)
```cpp
pair<int,int> a = {1, 5}, b = {1, 3};
cout << (b < a);     // 1 (true): firsts equal -> compare 3 < 5
cout << (a == b);    // 0
// This is WHY pairs sort nicely and work as keys.
```

### Nested pairs
```cpp
pair<int, pair<int,int>> edge = {10, {2, 3}};   // {weight, {u, v}}
cout << edge.first;            // 10
cout << edge.second.first;     // 2
cout << edge.second.second;    // 3
```

### Structured bindings (C++17) — clean unpacking
```cpp
pair<int,string> p = {1, "abc"};
auto [num, str] = p;           // num=1, str="abc"

// in loops over maps / vectors of pairs
vector<pair<int,int>> v = {{1,2},{3,4}};
for (auto& [x, y] : v) cout << x << "," << y << " ";   // 1,2 3,4
```

### swap & tie
```cpp
pair<int,int> a = {1,2}, b = {3,4};
swap(a, b);                    // a={3,4}, b={1,2}

int x, y;
tie(x, y) = make_pair(5, 6);   // x=5, y=6  (pre-C++17 unpacking)
tie(x, ignore) = p;            // grab first, discard second
```

---

## tuple — 3 or more values

### Construction & access
```cpp
tuple<int, int, string> t = {1, 2, "hi"};
auto t2 = make_tuple(1, 2.5, 'c', "str");   // mixed types

cout << get<0>(t);    // 1   (access by compile-time index)
cout << get<2>(t);    // "hi"
get<0>(t) = 99;       // mutable

cout << tuple_size<decltype(t)>::value;   // 3  (number of elements)
```

### Unpacking
```cpp
tuple<int,int,string> t = {1, 2, "hi"};

// C++17 structured binding (preferred)
auto [a, b, c] = t;            // a=1, b=2, c="hi"

// pre-C++17
int x, y; string s;
tie(x, y, s) = t;
tie(x, ignore, s) = t;         // skip the middle
```

### Comparison & sorting
```cpp
// tuples compare lexicographically across all members
vector<tuple<int,int,int>> v = {{1,2,3},{1,1,9},{0,5,5}};
sort(v.begin(), v.end());      // {0,5,5},{1,1,9},{1,2,3}

// build a tuple of references on the fly to sort by custom order
sort(v.begin(), v.end(), [](auto& a, auto& b){
    return tie(get<2>(a), get<0>(a)) < tie(get<2>(b), get<0>(b));
});
```

### tuple_cat — concatenate
```cpp
auto t1 = make_tuple(1, 2);
auto t2 = make_tuple("a", 'b');
auto big = tuple_cat(t1, t2);  // tuple<int,int,const char*,char>
```

---

## When to use what

| Need | Use |
|---|---|
| 2 values | `pair` |
| 3+ values | `tuple` (or a small `struct` for readability) |
| Coordinates (r, c) | `pair<int,int>` |
| Weighted edge | `pair<int, pair<int,int>>` or `tuple<int,int,int>` |
| Return multiple values from a function | `pair` / `tuple` / `struct` |

```cpp
// returning multiple values
pair<int,int> minmax_vals(vector<int>& v) {
    return {*min_element(v.begin(),v.end()), *max_element(v.begin(),v.end())};
}
auto [lo, hi] = minmax_vals(v);
```

---

## DSA use cases
- **Graphs:** `priority_queue<pair<int,int>>` for Dijkstra (`{dist, node}`); edge lists as `vector<tuple<int,int,int>>` (`{w,u,v}`).
- **Coordinates:** BFS/DFS on grids with `pair<int,int>` in a queue/stack.
- **Sorting by multiple keys:** pack keys into a tuple to get lexicographic ordering for free.
- **Map keys:** `map<pair<int,int>, int>` for memoizing on 2 indices.

## Gotchas
- ⚠️ `get<i>(t)` index must be a **compile-time constant** — you can't do `get<i>(t)` with a runtime `i`.
- ⚠️ `pair`/`tuple` member names (`first`, `second`, `get<0>`) hurt readability for domain data — a small `struct` with named fields is often clearer for non-throwaway code.
- For unordered_map keys you need a custom hash for `pair`/`tuple` (no default hash) — for ordered `map`/`set` they work out of the box.
