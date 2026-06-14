# Choosing the Right Container + Complexity Tables

A decision guide and the complexity tables to glance at before/while coding.

---

## Decision flowchart (read top to bottom, stop at first match)

```
Need key -> value mapping?
├── Need keys SORTED / range queries (lower_bound)?  -> map
└── Just fast lookup, order irrelevant?              -> unordered_map

Need a set of keys (no values)?
├── Need them SORTED / predecessor-successor?        -> set / multiset
└── Just "have I seen this?"                          -> unordered_set

Need to always pull the min or max next?              -> priority_queue
                                                         (or multiset if you also
                                                          remove arbitrary elements)

Order of processing matters?
├── First-In-First-Out (BFS)                          -> queue
├── Last-In-First-Out (DFS, parentheses, monotonic)   -> stack
└── Insert/remove at BOTH ends (sliding window)       -> deque

Need O(1) splice / stable iterators (LRU cache)?      -> list

Otherwise (default, 90% of problems)                  -> vector
```

---

## Master complexity table

| Container | Access `[i]` | Search | Insert | Erase | Min/Max | Ordered? |
|---|---|---|---|---|---|---|
| `vector` | O(1) | O(n) | O(1)* back / O(n) mid | O(n) | O(n) | insertion |
| `deque` | O(1) | O(n) | O(1) ends / O(n) mid | O(1) ends | O(n) | insertion |
| `list` | ❌ O(n) | O(n) | O(1)** | O(1)** | O(n) | insertion |
| `stack` | top O(1) | — | O(1) | O(1) | — | LIFO |
| `queue` | ends O(1) | — | O(1) | O(1) | — | FIFO |
| `priority_queue` | top O(1) | — | O(log n) | O(log n) top only | O(1) top | by priority |
| `set` / `map` | — | O(log n) | O(log n) | O(log n) | O(1) (begin/rbegin) | sorted |
| `multiset` / `multimap` | — | O(log n) | O(log n) | O(log n) | O(1) | sorted, dups |
| `unordered_set` / `unordered_map` | — | O(1) avg | O(1) avg | O(1) avg | O(n) | none |

\* amortized for `push_back`  \*\* given an iterator to the position

---

## Ordered vs Unordered — pick deliberately

| Question | Ordered (`set`/`map`) | Unordered (`unordered_*`) |
|---|---|---|
| Need sorted iteration? | ✅ | ❌ |
| Need `lower_bound`/floor/ceil/range? | ✅ | ❌ |
| Need fastest possible lookup? | ❌ (O(log n)) | ✅ (O(1) avg) |
| Worst-case guarantee? | ✅ O(log n) | ❌ O(n) on bad hash |
| Custom struct key — easy? | ✅ needs only `<` | ❌ needs a hash functor |

Rule of thumb: **default to `unordered_*` for speed; upgrade to ordered only when you need order or worst-case guarantees.**

---

## Common DSA task → container

| Task | Container(s) |
|---|---|
| Frequency count | `unordered_map<T,int>` (or `int[]` for small int keys) |
| Memoization | `unordered_map` / N-D `vector` |
| Two Sum / complement lookup | `unordered_map` / `unordered_set` |
| BFS (unweighted shortest path) | `queue` (+ `vector<bool> visited`) |
| DFS (iterative) | `stack` |
| Dijkstra / Prim | `priority_queue` of `{dist,node}` |
| Top-K largest | min-heap of size K |
| Running median | two heaps, or `multiset` |
| Sliding window max/min | `deque` (monotonic) |
| Sliding window with removable extremes | `multiset` |
| Next/Previous greater element | `stack` (monotonic) |
| Floor/ceil/nearest value | `set`/`map` + `lower_bound` |
| Interval scheduling / sweep line | sort + `priority_queue` / `multiset` |
| LRU cache | `list` + `unordered_map<key, iterator>` |
| Adjacency list (graph) | `vector<vector<int>>` |
| DP table | `vector` (1D/2D/3D) |
| Coordinate compression | `set` → vector → index map |
| Disjoint Set Union | plain `vector<int> parent, rank` |

---

## Memory & constant-factor notes
- `vector` has the best cache locality — fastest in practice for scans even when big-O ties.
- `unordered_*` and node-based `set`/`map`/`list` allocate per element → higher constant factor and memory.
- For dense small-integer keys, a raw array (`int cnt[N]`, `bool seen[N]`) beats any hash container.
- `deque` is chunked → slightly slower random access than `vector`.

## Final reminders (the 5 universal gotchas)
1. `stack/queue/priority_queue.pop()` returns **void** — read top/front first.
2. `map[key]`/`unordered_map[key]` **auto-creates** keys — use `find`/`count`/`contains` to check.
3. `multiset/multimap.erase(value)` removes **all** copies — use `erase(find(value))` for one.
4. `priority_queue` default is **max**-heap; `greater<>` → min-heap.
5. `find`/`lower_bound`/`begin`/`max_element` return **iterators** (deref `*`, check `end()`); `front`/`back`/`top`/`at` return **values**.
