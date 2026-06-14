# Iterators — How STL Containers Are Traversed

An iterator is a generalized "pointer" into a container. STL algorithms work on iterator **ranges `[begin, end)`** — `end` points one **past** the last element (not a real element). Understanding iterators clears up most STL confusion (especially "iterator vs value").

---

## begin / end and friends
```cpp
vector<int> v = {10, 20, 30};

v.begin()    // points to first element (10)
v.end()      // points PAST the last (NOT a valid element — never dereference)
v.rbegin()   // reverse: last element (30)
v.rend()     // reverse: before-first (sentinel)
v.cbegin()   // const iterator (read-only)

auto it = v.begin();
cout << *it;        // 10   — DEREFERENCE to read the value
++it;               // move to next (20)
cout << *it;        // 20
```

For maps/pairs, dereferencing gives a pair — use `->`:
```cpp
map<int,string> m = {{1,"a"}};
auto it = m.begin();
cout << it->first << " " << it->second;   // 1 a
cout << (*it).first;                       // same thing
```

---

## Iterator vs value — the #1 confusion

| Function returns ITERATOR | Function returns VALUE |
|---|---|
| points to *where* — deref with `*` | the data itself — use directly |
| compare to `.end()` for "not found" | — |
| `find`, `lower_bound`, `begin`, `max_element`, `insert`, `erase` | `front`, `back`, `top`, `at`, `[]`, `size`, `count` |

```cpp
auto it = s.find(2);        // ITERATOR -> *it to read, check it != s.end()
auto x  = v.back();         // VALUE    -> use x directly

// how to tell with `auto`: think about the FUNCTION, not the variable.
// "locate/search/where"  -> iterator (can return end() = not found)
// "give me the element"   -> value
```
Quick proof when unsure: `int x = container.func();` — if it fails to compile, `func()` returned an iterator.

---

## Moving iterators
```cpp
auto it = v.begin();

++it;  it++;             // forward one
--it;  it--;             // back one (bidirectional containers)
it += 3;                 // jump 3 (RANDOM-ACCESS iterators only: vector/deque/string)

advance(it, 5);          // move forward 5 (works on ANY iterator; O(k) for list/set)
auto nx = next(it, 2);   // iterator 2 ahead (returns new, doesn't modify it)
auto pv = prev(it, 1);   // iterator 1 behind

int idx = it - v.begin();           // iterator -> index (random-access only)
int dist = distance(a, b);          // # steps from a to b (any iterator)
```
> ⚠️ `it + 3`, `it - it2`, `it[i]` work ONLY for **random-access** iterators (`vector`, `deque`, `string`, arrays). For `set`/`map`/`list` use `advance`/`next`/`prev`.

---

## Iterator categories (what each container gives you)
| Category | Supports | Containers |
|---|---|---|
| Random-access | `+n`, `-n`, `[]`, `<` | `vector`, `deque`, `string`, array |
| Bidirectional | `++`, `--` | `list`, `set`, `map`, `multiset`, `multimap` |
| Forward | `++` only | `forward_list`, `unordered_*` |
| Input/Output | single-pass | streams |

This is why `sort(s.begin(), s.end())` fails on a `set` (needs random-access) — use the container's own ordering instead.

---

## Erase while iterating (critical pattern)

`erase` invalidates the erased iterator. The safe idiom uses the returned next-iterator:

```cpp
// map / set / list: erase returns the next valid iterator
for (auto it = m.begin(); it != m.end(); ) {
    if (shouldRemove(it->second)) it = m.erase(it);   // advance via return value
    else                          ++it;
}

// vector: same idea, but prefer erase-remove for bulk deletes
v.erase(remove_if(v.begin(), v.end(), pred), v.end());
```
> ⚠️ Do NOT write `for (auto it=...; ...; ++it) m.erase(it);` — after `erase(it)`, `it` is dangling and `++it` is UB.

---

## Iterator invalidation rules (know these for interviews)
| Container | Insert invalidates | Erase invalidates |
|---|---|---|
| `vector` | ALL iters if realloc; else those after pos | erased + everything after |
| `deque` | all (usually) | all (usually) |
| `list` / `forward_list` | none | only the erased element |
| `set`/`map`/`multi*` | none | only the erased element |
| `unordered_*` | all iters on **rehash**; refs stay valid | only the erased element |

Takeaway: `list`/`set`/`map` keep other iterators valid across insert/erase — that's why LRU cache stores `list` iterators in a map.

---

## Inserter iterators (for algorithm output)
```cpp
vector<int> out;
copy(a.begin(), a.end(), back_inserter(out));     // push_back each
set<int> s;
copy(a.begin(), a.end(), inserter(s, s.end()));   // insert into set
// front_inserter(dq) for push_front
```
Without an inserter, algorithms like `copy`/`transform` assume the destination already has enough space.

---

## Range-based for is iterators underneath
```cpp
for (int x : v)         { }   // copy each element (read)
for (int& x : v)        { }   // reference -> can MODIFY
for (const auto& x : v) { }   // no copy, read-only (best for big objects/strings)
for (auto& [k, val] : m){ }   // structured binding over a map
```
> ⚠️ Use `&` to modify or to avoid copying large elements (`string`, `vector`, structs).

## Top gotchas
- ⚠️ `*end()` is undefined — `end()` is a sentinel, never a value.
- ⚠️ `auto` hides whether you have an iterator or a value — reason from the **function name**.
- ⚠️ Don't `++it` after `erase(it)` on the same line for node-based containers — use the return value.
- ⚠️ Random-access ops (`it+n`, `it[i]`) only on vector/deque/string.
