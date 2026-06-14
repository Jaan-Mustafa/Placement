# C++ STL — Complete Reference for DSA

A file-per-topic deep dive into the C++ Standard Template Library, focused on what you use to solve DSA / interview problems. Every file lists the container's full function set, time complexities, gotchas, and runnable examples.

## How to read these files

- **Header note:** in contests use `#include <bits/stdc++.h>` + `using namespace std;` to get everything.
- Each function has its **time complexity** and a working example.
- ⚠️ marks the gotchas that cause bugs / WA / TLE in interviews.

## Files

| # | File | Container / Topic | Use it for |
|---|---|---|---|
| 01 | [01_Vector.md](01_Vector.md) | `vector` | Dynamic array — the default container |
| 02 | [02_String.md](02_String.md) | `string` | Text processing |
| 03 | [03_Pair_and_Tuple.md](03_Pair_and_Tuple.md) | `pair`, `tuple` | Grouping 2–3 values |
| 04 | [04_Stack.md](04_Stack.md) | `stack` | LIFO, monotonic stack, DFS, parentheses |
| 05 | [05_Queue.md](05_Queue.md) | `queue` | FIFO, BFS, level-order |
| 06 | [06_Deque.md](06_Deque.md) | `deque` | Double-ended, sliding window max |
| 07 | [07_Priority_Queue.md](07_Priority_Queue.md) | `priority_queue` | Heaps, Dijkstra, top-K |
| 08 | [08_Set_and_Multiset.md](08_Set_and_Multiset.md) | `set`, `multiset` | Sorted unique / dup keys, ordered queries |
| 09 | [09_Unordered_Set.md](09_Unordered_Set.md) | `unordered_set` | O(1) presence checks |
| 10 | [10_Map_and_Multimap.md](10_Map_and_Multimap.md) | `map`, `multimap` | Sorted key→value |
| 11 | [11_Unordered_Map.md](11_Unordered_Map.md) | `unordered_map` | O(1) key→value, freq counts, memoization |
| 12 | [12_List_and_Forward_List.md](12_List_and_Forward_List.md) | `list`, `forward_list` | Linked lists, LRU cache |
| 13 | [13_Algorithms.md](13_Algorithms.md) | `<algorithm>`, `<numeric>` | sort, binary search, accumulate, permutations |
| 14 | [14_Iterators.md](14_Iterators.md) | Iterators | begin/end, dereferencing, invalidation |
| 15 | [15_Choosing_Container.md](15_Choosing_Container.md) | Decision guide | Which container to pick & complexity table |

## The 5 gotchas that bite everyone

1. `stack.pop()` / `queue.pop()` **return nothing** — read with `top()`/`front()` first.
2. `map[key]` / `unordered_map[key]` **auto-creates** the key — use `find`/`count` to just check.
3. `multiset.erase(x)` removes **all** copies — use `erase(find(x))` for one.
4. `priority_queue<int>` is a **max-heap**; `greater<int>` makes it a **min-heap**.
5. Functions like `find`, `lower_bound`, `begin` return **iterators** (dereference with `*`); `front`, `back`, `top`, `at` return **values**.
