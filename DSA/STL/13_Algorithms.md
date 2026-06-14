# `<algorithm>` & `<numeric>` — The Function Toolbox

`#include <algorithm>` · `#include <numeric>`

These operate on **iterator ranges `[first, last)`** — `last` points one past the end. Most return either an iterator (dereference with `*`) or a value. Learn these and you stop hand-rolling loops.

---

## Sorting & ordering

```cpp
sort(v.begin(), v.end());                        // ascending, O(n log n)
sort(v.begin(), v.end(), greater<int>());        // descending
sort(v.begin(), v.end(), [](int a, int b){ return a > b; });   // custom

// sort vector of pairs by SECOND, tie-break by first
sort(v.begin(), v.end(), [](auto& a, auto& b){
    if (a.second != b.second) return a.second < b.second;
    return a.first < b.first;
});

stable_sort(v.begin(), v.end());                 // keeps relative order of equals
partial_sort(v.begin(), v.begin()+k, v.end());   // smallest k sorted at front, O(n log k)
nth_element(v.begin(), v.begin()+k, v.end());    // kth element in place, O(n) avg
is_sorted(v.begin(), v.end());                   // bool
reverse(v.begin(), v.end());                     // reverse range
rotate(v.begin(), v.begin()+k, v.end());         // left-rotate by k
```

> ⚠️ The comparator must be a **strict weak ordering** — use `<` (return false for equal elements). Using `<=` causes UB / crashes.

---

## Binary search — range MUST be sorted

```cpp
binary_search(v.begin(), v.end(), x);            // bool: present?

// lower_bound: first position where element >= x
auto lb = lower_bound(v.begin(), v.end(), x);
int idx = lb - v.begin();                         // convert iterator -> index

// upper_bound: first position where element > x
auto ub = upper_bound(v.begin(), v.end(), x);

// number of occurrences of x in sorted v
int cnt = upper_bound(v.begin(),v.end(),x) - lower_bound(v.begin(),v.end(),x);

// equal_range: {lower_bound, upper_bound} together
auto [lo, hi] = equal_range(v.begin(), v.end(), x);
```

> On `vector` these are O(log n). On `set`/`map`, call the **member** versions instead (free functions are O(n) there).

---

## Min / max

```cpp
*max_element(v.begin(), v.end());                // largest VALUE (note the *)
*min_element(v.begin(), v.end());                // smallest value
auto [mnIt, mxIt] = minmax_element(v.begin(), v.end());   // both in one pass

max(a, b);  min(a, b);                            // two values
max({a, b, c, d});  min({a, b, c, d});            // init-list of values
auto [lo, hi] = minmax(a, b);                     // {smaller, larger}

int idx = max_element(v.begin(), v.end()) - v.begin();    // index of max
```

> ⚠️ `max_element` returns an **iterator** → use `*`. `max(a,b)` returns a **value** → no `*`.

---

## Numeric (`<numeric>`)

```cpp
int sum = accumulate(v.begin(), v.end(), 0);            // sum, init = 0
long long s = accumulate(v.begin(), v.end(), 0LL);      // avoid int overflow!
int prod = accumulate(v.begin(), v.end(), 1, multiplies<int>());

// custom fold
int maxv = accumulate(v.begin(), v.end(), INT_MIN,
                      [](int acc, int x){ return max(acc, x); });

partial_sum(v.begin(), v.end(), prefix.begin());        // prefix sums
adjacent_difference(v.begin(), v.end(), diff.begin());  // diffs of neighbors
iota(v.begin(), v.end(), 0);                            // fill 0,1,2,3,...
int dot = inner_product(a.begin(), a.end(), b.begin(), 0);   // dot product
int g = gcd(12, 18);   int l = lcm(4, 6);               // C++17
```

> ⚠️ The init value's **type** decides the accumulation type. `accumulate(v,v,0)` on big values overflows `int` — pass `0LL`.

---

## Counting & predicates

```cpp
count(v.begin(), v.end(), x);                    // # elements == x
count_if(v.begin(), v.end(), [](int x){ return x % 2 == 0; });   // # matching

find(v.begin(), v.end(), x);                     // iterator to first x (or end())
find_if(v.begin(), v.end(), pred);               // first matching
find_if_not(v.begin(), v.end(), pred);

all_of(v.begin(), v.end(), pred);                // bool: all match?
any_of(v.begin(), v.end(), pred);                // bool: at least one?
none_of(v.begin(), v.end(), pred);               // bool: zero match?

adjacent_find(v.begin(), v.end());               // first pair of equal neighbors
search(v.begin(), v.end(), sub.begin(), sub.end());   // find a subsequence
```

---

## Modifying / transforming

```cpp
fill(v.begin(), v.end(), 0);                     // set all to 0
fill_n(v.begin(), 3, 9);                          // first 3 -> 9
generate(v.begin(), v.end(), [](){ return 0; }); // fill from a generator
copy(a.begin(), a.end(), b.begin());             // copy a into b
copy_if(a.begin(), a.end(), back_inserter(b), pred);   // filtered copy

transform(v.begin(), v.end(), v.begin(), [](int x){ return x*x; });  // map in place
transform(a.begin(), a.end(), b.begin(), out.begin(), plus<int>());  // a[i]+b[i]

replace(v.begin(), v.end(), 0, -1);              // replace all 0 with -1
swap(a, b);                                       // swap two objects
iter_swap(it1, it2);                              // swap pointed-to elements

// remove (doesn't resize!) -> pair with erase
v.erase(remove(v.begin(), v.end(), x), v.end()); // delete all x
v.erase(remove_if(v.begin(), v.end(), pred), v.end());

// unique consecutive dups -> sort first to dedupe fully
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());
```

> ⚠️ `remove`/`unique` do **not** change the container size — they shift elements and return a new logical end. Always pair with `erase` (erase-remove idiom).

---

## Permutations & combinations

```cpp
sort(v.begin(), v.end());
do {
    // process this permutation
} while (next_permutation(v.begin(), v.end()));   // all perms in lex order

prev_permutation(v.begin(), v.end());             // previous lex permutation
```

---

## Set operations (on sorted ranges)
```cpp
set_union(a.begin(),a.end(), b.begin(),b.end(), back_inserter(out));
set_intersection(a.begin(),a.end(), b.begin(),b.end(), back_inserter(out));
set_difference(a.begin(),a.end(), b.begin(),b.end(), back_inserter(out));
includes(a.begin(),a.end(), b.begin(),b.end());   // is b a subset of a?
merge(a.begin(),a.end(), b.begin(),b.end(), out.begin());   // merge two sorted
```

---

## Heap algorithms (turn a vector into a heap manually)
```cpp
make_heap(v.begin(), v.end());                   // build max-heap, O(n)
push_heap(v.begin(), v.end());                   // after push_back, sift up
pop_heap(v.begin(), v.end());                    // move max to back; then pop_back
sort_heap(v.begin(), v.end());                   // heapsort the heap
is_heap(v.begin(), v.end());                     // bool
```

---

## Bit tricks (GCC builtins — handy in DSA)
```cpp
__builtin_popcount(x);       // # of set bits (int)
__builtin_popcountll(x);     // for long long
__builtin_clz(x);            // leading zeros  -> 31 - log2(x) for int
__builtin_ctz(x);            // trailing zeros (index of lowest set bit)
__lg(x);                     // floor(log2(x))

// iterate over subsets of a bitmask
for (int sub = mask; sub; sub = (sub - 1) & mask) { /* sub is a subset */ }
```

---

## Cheat sheet: returns iterator vs value
| Returns ITERATOR (use `*`) | Returns VALUE (use directly) |
|---|---|
| `find`, `find_if` | `count`, `count_if` |
| `lower_bound`, `upper_bound` | `binary_search` (bool) |
| `max_element`, `min_element` | `max`, `min`, `accumulate` |
| `remove`, `unique` (new end) | `all_of/any_of/none_of` (bool) |
| `begin`, `end` | `is_sorted` (bool) |

## Top gotchas
- ⚠️ `remove`/`unique` don't resize — pair with `erase`.
- ⚠️ `accumulate` init type controls overflow — use `0LL` for big sums.
- ⚠️ `*max_element(...)` needs the `*`; `max(a,b)` does not.
- ⚠️ Binary-search family requires a **sorted** range.
- ⚠️ Comparators must be strict-weak (`<`, not `<=`).
