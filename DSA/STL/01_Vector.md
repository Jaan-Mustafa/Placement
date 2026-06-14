# vector — Dynamic Array

`#include <vector>`

A contiguous, resizable array. **The default container** — random access in O(1), amortized O(1) append. Internally keeps a block of memory with a `size` (used) and `capacity` (allocated); when full it reallocates to a bigger block (usually 2×) and copies.

---

## Construction

```cpp
vector<int> v;                      // empty
vector<int> v(5);                   // {0,0,0,0,0}  (5 default-initialized)
vector<int> v(5, 9);                // {9,9,9,9,9}  (5 copies of 9)
vector<int> v = {1, 2, 3};          // initializer list
vector<int> v2(v);                  // copy of v
vector<int> v3(v.begin(), v.end()); // from a range
vector<int> v4(arr, arr + n);       // from a C array

// 2D
vector<vector<int>> grid(n, vector<int>(m, 0));   // n x m filled with 0
vector<vector<int>> adj(n);                        // n empty rows (graph adj list)

// 3D
vector<vector<vector<int>>> dp(a, vector<vector<int>>(b, vector<int>(c, -1)));
```

---

## Full function list

### Element access
| Function | Description | Complexity |
|---|---|---|
| `v[i]` | Access (no bounds check — fast) | O(1) |
| `v.at(i)` | Access with bounds check (throws `out_of_range`) | O(1) |
| `v.front()` | First element (value) | O(1) |
| `v.back()` | Last element (value) | O(1) |
| `v.data()` | Pointer to underlying array | O(1) |

### Capacity
| Function | Description | Complexity |
|---|---|---|
| `v.size()` | Number of elements | O(1) |
| `v.empty()` | `true` if size == 0 | O(1) |
| `v.capacity()` | Allocated slots (≥ size) | O(1) |
| `v.reserve(n)` | Pre-allocate capacity for n (avoids reallocs) | O(n) |
| `v.shrink_to_fit()` | Free unused capacity | O(n) |
| `v.max_size()` | Theoretical max elements | O(1) |

### Modifiers
| Function | Description | Complexity |
|---|---|---|
| `v.push_back(x)` | Append (copy) | Amortized O(1) |
| `v.emplace_back(args)` | Construct element in place at end | Amortized O(1) |
| `v.pop_back()` | Remove last (returns nothing) | O(1) |
| `v.insert(pos, x)` | Insert before iterator `pos` | O(n) |
| `v.insert(pos, cnt, x)` | Insert `cnt` copies | O(n) |
| `v.insert(pos, first, last)` | Insert a range | O(n) |
| `v.emplace(pos, args)` | Construct in place at pos | O(n) |
| `v.erase(pos)` | Erase one element | O(n) |
| `v.erase(first, last)` | Erase a range `[first, last)` | O(n) |
| `v.clear()` | Remove all (size→0, capacity kept) | O(n) |
| `v.resize(n)` | Grow/shrink to n (new slots default-init) | O(n) |
| `v.resize(n, x)` | Grow filling new slots with x | O(n) |
| `v.assign(n, x)` | Replace contents with n copies of x | O(n) |
| `v.assign(first, last)` | Replace contents with a range | O(n) |
| `v.swap(v2)` | Swap two vectors (O(1) — swaps pointers) | O(1) |

### Iterators
| Function | Returns |
|---|---|
| `v.begin()` / `v.end()` | forward iterators |
| `v.rbegin()` / `v.rend()` | reverse iterators |
| `v.cbegin()` / `v.cend()` | const iterators |

---

## Examples

### Basics
```cpp
vector<int> v = {3, 1, 4, 1, 5};

v.push_back(9);                  // {3,1,4,1,5,9}
v.pop_back();                    // {3,1,4,1,5}
cout << v.front() << v.back();   // 3 5
cout << v.size();                // 5

// iterate (3 ways)
for (int x : v) cout << x << " ";                 // range-for (read)
for (int& x : v) x *= 2;                          // range-for (modify, note &)
for (int i = 0; i < v.size(); i++) cout << v[i];  // index
for (auto it = v.begin(); it != v.end(); ++it) cout << *it;  // iterator
```

### Insert and erase
```cpp
vector<int> v = {10, 20, 30, 40};

v.insert(v.begin() + 1, 99);     // {10,99,20,30,40}
v.insert(v.end(), 2, 7);         // {10,99,20,30,40,7,7}

v.erase(v.begin());              // erase index 0 -> {99,20,30,40,7,7}
v.erase(v.begin() + 1, v.begin() + 3);   // erase [1,3) -> {99,40,7,7}
```

### Erase-remove idiom (delete by value)
```cpp
vector<int> v = {1, 2, 2, 3, 2, 4};
// remove() shifts non-matching to front, returns new logical end
v.erase(remove(v.begin(), v.end(), 2), v.end());   // {1,3,4}

// erase_if (C++20) — one-liner
erase(v, 3);                     // remove all 3s
erase_if(v, [](int x){ return x % 2 == 0; });   // remove evens
```

### Sorting & searching
```cpp
vector<int> v = {3, 1, 4, 1, 5};
sort(v.begin(), v.end());                  // {1,1,3,4,5}
sort(v.rbegin(), v.rend());                // descending {5,4,3,1,1}
reverse(v.begin(), v.end());

// dedupe a sorted vector
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());

// max / min / sum
int mx = *max_element(v.begin(), v.end());
int mn = *min_element(v.begin(), v.end());
int sum = accumulate(v.begin(), v.end(), 0);
```

### 2D vector iteration
```cpp
vector<vector<int>> g = {{1,2,3},{4,5,6}};
for (auto& row : g)
    for (int x : row) cout << x << " ";

cout << g.size();       // 2 (rows)
cout << g[0].size();    // 3 (cols)
```

---

## Performance notes & gotchas

- ⚠️ **`reserve` when you know the size.** Without it, repeated `push_back` triggers reallocation + copy. `v.reserve(n)` makes n appends truly O(1) total.
- ⚠️ **`emplace_back` vs `push_back`:** `emplace_back` constructs in place (no temporary). For `push_back(MyStruct{a,b})` vs `emplace_back(a,b)`, the latter avoids a copy. For ints there's no difference.
- ⚠️ **Iterator invalidation:** any reallocation (`push_back` past capacity, `insert`, `resize`) invalidates ALL existing iterators/pointers/references. `erase` invalidates everything at/after the erase point. Don't hold an old iterator across these.
- ⚠️ **`vector<bool>` is special** — it's a bit-packed proxy, NOT a real array of bools. You can't take `&v[i]`. Use `vector<char>` or `deque<bool>` if you need normal behavior.
- **`clear()` keeps capacity.** To actually free memory: `v.clear(); v.shrink_to_fit();` or `vector<int>().swap(v);`.
- **Passing to functions:** pass by `const vector<int>&` (read) or `vector<int>&` (modify) to avoid copying the whole array.

## DSA use cases
- Default storage for almost everything.
- Adjacency list for graphs: `vector<vector<int>> adj(n);`
- DP tables: `vector<vector<int>> dp(...)`.
- Prefix sums, difference arrays, frequency arrays.
