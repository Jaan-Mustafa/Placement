# deque — Double-Ended Queue

`#include <deque>`

A "double-ended queue": O(1) insert/remove at **both** ends AND O(1) random access. Internally a sequence of fixed-size blocks with a map of block pointers (not one contiguous buffer like vector). The most flexible sequence container.

---

## Construction
```cpp
deque<int> dq;                      // empty
deque<int> dq(5);                   // {0,0,0,0,0}
deque<int> dq(5, 9);                // {9,9,9,9,9}
deque<int> dq = {1, 2, 3};          // init list
```

---

## Full function list

### Access
| Function | Description | Complexity |
|---|---|---|
| `dq[i]` / `dq.at(i)` | Random access | O(1) |
| `dq.front()` / `dq.back()` | Ends | O(1) |

### Modify
| Function | Description | Complexity |
|---|---|---|
| `dq.push_back(x)` / `dq.emplace_back` | Insert at back | O(1) |
| `dq.push_front(x)` / `dq.emplace_front` | Insert at front | O(1) |
| `dq.pop_back()` | Remove from back | O(1) |
| `dq.pop_front()` | Remove from front | O(1) |
| `dq.insert(pos, x)` | Insert anywhere | O(n) |
| `dq.erase(pos)` / `erase(l,r)` | Erase element / range | O(n) |
| `dq.clear()` | Remove all | O(n) |
| `dq.resize(n)` / `dq.assign(...)` | Like vector | O(n) |

### Capacity / iterators
| Function | Description |
|---|---|
| `dq.size()` / `dq.empty()` | size / emptiness |
| `dq.begin()/end()/rbegin()/rend()` | full iterator support (unlike stack/queue) |

> Note: deque has **no** `capacity()`/`reserve()` (no single buffer to reserve).

---

## Examples

### Basics — both ends
```cpp
deque<int> dq;
dq.push_back(2);     // {2}
dq.push_front(1);    // {1,2}
dq.push_back(3);     // {1,2,3}
dq.push_front(0);    // {0,1,2,3}

cout << dq.front() << " " << dq.back();   // 0 3
cout << dq[2];                            // 2 (random access!)

dq.pop_front();      // {1,2,3}
dq.pop_back();       // {1,2}

for (int x : dq) cout << x << " ";        // 1 2  (iterable!)
```

---

## The killer DSA pattern: Sliding Window Maximum

Find the max in every window of size k, in **O(n)** total, using a **monotonic deque** that stores indices in decreasing order of value.

```cpp
vector<int> maxSlidingWindow(vector<int>& a, int k) {
    deque<int> dq;                     // stores INDICES, values decreasing
    vector<int> res;
    for (int i = 0; i < a.size(); i++) {
        // 1. drop indices that fell out of the window on the left
        if (!dq.empty() && dq.front() <= i - k)
            dq.pop_front();

        // 2. drop smaller values from the back — they can never be the max
        while (!dq.empty() && a[dq.back()] <= a[i])
            dq.pop_back();

        // 3. add current index
        dq.push_back(i);

        // 4. front is the max of the current window
        if (i >= k - 1)
            res.push_back(a[dq.front()]);
    }
    return res;
}
// a = [1,3,-1,-3,5,3,6,7], k = 3  ->  [3,3,5,5,6,7]
```

Why deque: we need to remove stale elements from the **front** (out of window) and smaller elements from the **back** (dominated) — both O(1) only a deque gives.

### Sliding window minimum
Same code, flip the comparison in step 2 to `a[dq.back()] >= a[i]`.

### Palindrome check using both ends
```cpp
bool isPalindrome(string s) {
    deque<char> dq(s.begin(), s.end());
    while (dq.size() > 1) {
        if (dq.front() != dq.back()) return false;
        dq.pop_front();
        dq.pop_back();
    }
    return true;
}
```

### 0-1 BFS (deque instead of priority_queue)
```cpp
// edges have weight 0 or 1: push_front for weight-0, push_back for weight-1
deque<int> dq;
dq.push_front(start);
dist[start] = 0;
while (!dq.empty()) {
    int u = dq.front(); dq.pop_front();
    for (auto [v, w] : adj[u]) {
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            if (w == 0) dq.push_front(v);
            else        dq.push_back(v);
        }
    }
}
```

---

## deque vs vector
| | vector | deque |
|---|---|---|
| `push_back` | amortized O(1) | O(1) |
| `push_front` | O(n) ❌ | O(1) ✅ |
| Random access `[i]` | O(1), slightly faster | O(1), slight overhead |
| Memory layout | one contiguous block | chunked blocks |
| `reserve`/`capacity` | yes | no |
| Cache locality | better | worse |

Use `vector` by default; use `deque` when you genuinely need fast **front** operations.

## When to use a deque
- **Sliding window max/min** (monotonic deque) — the signature use.
- Need push/pop at both ends (BFS variants, 0-1 BFS, palindrome).
- A double-ended buffer / scrollback.

## Gotchas
- ⚠️ Slightly worse cache performance than vector — don't reach for it unless you need front ops.
- ⚠️ `insert`/`erase` in the **middle** is still O(n).
- In the monotonic-deque pattern, store **indices** (not values) so you can check the window boundary.
