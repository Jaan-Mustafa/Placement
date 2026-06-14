# list & forward_list — Linked Lists

`#include <list>` (doubly linked) · `#include <forward_list>` (singly linked)

`std::list` is a **doubly linked list**: O(1) insert/erase **anywhere given an iterator**, but no random access (`l[i]` doesn't exist) and poor cache locality. Mostly useful when you need O(1) splice/move of elements — the classic case is an **LRU cache**.

---

## list construction
```cpp
list<int> l;                        // empty
list<int> l = {1, 2, 3};
list<int> l2(5, 0);                 // {0,0,0,0,0}
list<int> l3(v.begin(), v.end());   // from a range
```

## Full function list (list)

### Access
| Function | Description | Complexity |
|---|---|---|
| `l.front()` / `l.back()` | First / last element | O(1) |
| (no `[]`, no `at`) | — random access NOT supported | — |

### Modify
| Function | Description | Complexity |
|---|---|---|
| `l.push_back(x)` / `l.push_front(x)` | Insert at end / start | O(1) |
| `l.pop_back()` / `l.pop_front()` | Remove from end / start | O(1) |
| `l.insert(it, x)` | Insert before iterator | O(1) |
| `l.erase(it)` | Erase at iterator | O(1) |
| `l.clear()` | Remove all | O(n) |
| `l.resize(n)` / `l.assign(...)` | Like other containers | O(n) |

### List-specific operations (these are member functions for a reason)
| Function | Description | Complexity |
|---|---|---|
| `l.splice(pos, l2)` | **Move** all of l2 into l before pos — no copy | O(1) |
| `l.splice(pos, l2, it)` | Move one element from l2 | O(1) |
| `l.remove(x)` | Remove all elements == x | O(n) |
| `l.remove_if(pred)` | Remove all matching predicate | O(n) |
| `l.unique()` | Remove **consecutive** duplicates | O(n) |
| `l.merge(l2)` | Merge two **sorted** lists | O(n) |
| `l.sort()` | Sort (can't use `std::sort` — no random access) | O(n log n) |
| `l.reverse()` | Reverse in place | O(n) |

> ⚠️ Use the **member** `l.sort()` / `l.reverse()` / `l.unique()`, NOT the `<algorithm>` versions — those need random-access iterators that `list` doesn't provide.

---

## Examples

### Basics
```cpp
list<int> l = {1, 2, 3};
l.push_front(0);          // {0,1,2,3}
l.push_back(4);           // {0,1,2,3,4}
l.pop_front();            // {1,2,3,4}
cout << l.front() << l.back();   // 1 4

for (int x : l) cout << x << " ";   // iterable (bidirectional)

auto it = l.begin();
advance(it, 2);           // point to 3rd element (no l[2]!)
l.insert(it, 99);         // O(1) insert -> {1,2,99,3,4}
l.erase(it);              // O(1) erase the 3
```

### Member algorithms
```cpp
list<int> l = {3, 1, 2, 1, 3};
l.sort();                 // {1,1,2,3,3}
l.unique();               // {1,2,3}   (removes CONSECUTIVE dups — sort first)
l.reverse();              // {3,2,1}
l.remove(2);              // {3,1}
```

### splice — O(1) move (the LRU superpower)
```cpp
list<int> a = {1, 2, 3}, b = {7, 8};
auto it = a.begin(); advance(it, 1);
a.splice(it, b);          // move all of b before a[1]:  a = {1,7,8,2,3}, b = {}
```

---

## DSA pattern: LRU Cache (list + unordered_map)

`list` stores keys in usage order (most-recent at front); the map gives O(1) lookup of each key's list node so we can move it to front in O(1) via `splice`.

```cpp
class LRUCache {
    int cap;
    list<pair<int,int>> dll;                              // {key, value}, front = most recent
    unordered_map<int, list<pair<int,int>>::iterator> mp; // key -> node iterator
public:
    LRUCache(int c) : cap(c) {}

    int get(int key) {
        if (!mp.count(key)) return -1;
        // move accessed node to front in O(1)
        dll.splice(dll.begin(), dll, mp[key]);
        return mp[key]->second;
    }

    void put(int key, int value) {
        if (mp.count(key)) {
            mp[key]->second = value;
            dll.splice(dll.begin(), dll, mp[key]);        // refresh recency
            return;
        }
        if (dll.size() == cap) {                          // evict least-recent (back)
            mp.erase(dll.back().first);
            dll.pop_back();
        }
        dll.push_front({key, value});
        mp[key] = dll.begin();
    }
};
```
This is the canonical interview answer — every op is O(1) because `splice` moves a node without invalidating its iterator (unlike vector/deque).

---

## forward_list (singly linked)
Lower memory (one pointer per node), but only forward iteration and `_after` operations:
```cpp
forward_list<int> fl = {1, 2, 3};
fl.push_front(0);                 // only push_front (no push_back)
fl.insert_after(fl.begin(), 99);  // insert AFTER an iterator
fl.erase_after(fl.begin());       // erase the element after
fl.pop_front();
// also: sort(), reverse(), merge(), unique(), remove()
```
Rarely needed in interviews — mentioned for completeness.

---

## When to use list
- LRU/LFU cache (O(1) splice with a map of iterators).
- Frequent insertion/deletion in the **middle** when you already hold an iterator.
- When iterators/references must stay valid after insert/erase elsewhere (list never invalidates other nodes).

## When NOT to use
- ⚠️ If you need random access or index math → `vector`/`deque`.
- ⚠️ For most "linked list" interview problems (reverse list, detect cycle) you implement your **own** `struct ListNode { int val; ListNode* next; };` — those expect a raw pointer structure, not `std::list`.

## Gotchas
- ⚠️ No `[]` / `at` — use `advance(it, k)` to walk to a position (O(k)).
- ⚠️ `unique()` only removes **consecutive** duplicates — `sort()` first to remove all.
- ⚠️ Use member `sort/reverse/unique`, not the `<algorithm>` free functions.
- ⚠️ `splice` is O(1) and moves nodes (source loses them) — that's the whole point.
