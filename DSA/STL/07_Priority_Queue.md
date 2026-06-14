# priority_queue — Heap

`#include <queue>`

A binary heap exposed as a container adaptor (backed by `vector` + heap algorithms). Gives you the **highest-priority element in O(1)** and insert/remove in O(log n). **Max-heap by default.**

---

## The 3 declarations you must memorize
```cpp
// 1. MAX-heap (largest on top) — DEFAULT
priority_queue<int> maxHeap;

// 2. MIN-heap (smallest on top) — note greater<int>
priority_queue<int, vector<int>, greater<int>> minHeap;

// 3. Custom comparator
auto cmp = [](int a, int b){ return a > b; };   // min-heap
priority_queue<int, vector<int>, decltype(cmp)> pq(cmp);
```

> ⚠️ The comparator is "reversed" from intuition: `greater<int>` → **min**-heap, `less<int>` (default) → **max**-heap. The comparator answers "does a have LOWER priority than b?" — whatever is NOT lower bubbles to the top.

---

## Full function list
| Function | Description | Complexity |
|---|---|---|
| `pq.push(x)` | Insert | O(log n) |
| `pq.emplace(args)` | Construct in place + insert | O(log n) |
| `pq.pop()` | Remove top — **returns void** | O(log n) |
| `pq.top()` | View top (max or min) — **const ref** | O(1) |
| `pq.size()` | Number of elements | O(1) |
| `pq.empty()` | True if empty | O(1) |
| `pq.swap(pq2)` | Swap two heaps | O(1) |

> No iteration, no `clear()`, no decrease-key. `top()` is read-only (can't modify in place).

---

## Examples

### Max-heap & min-heap
```cpp
priority_queue<int> mx;
mx.push(3); mx.push(1); mx.push(5);
cout << mx.top();      // 5 (largest)
mx.pop();              // removes 5, top -> 3

priority_queue<int, vector<int>, greater<int>> mn;
mn.push(3); mn.push(1); mn.push(5);
cout << mn.top();      // 1 (smallest)
```

### Build a heap from a vector in O(n)
```cpp
vector<int> v = {3, 1, 4, 1, 5};
priority_queue<int> pq(v.begin(), v.end());   // heapify, O(n) not O(n log n)
```

### Heap of pairs (sorts by first, then second)
```cpp
priority_queue<pair<int,int>> pq;     // MAX-heap on .first
pq.push({2, 100});
pq.push({5, 1});
pq.push({5, 9});
cout << pq.top().first;               // 5 (then larger .second = 9 wins)

// min-heap of pairs (Dijkstra: {dist, node})
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> minpq;
```

### Custom comparator — min-heap by the SECOND element
```cpp
auto cmp = [](pair<int,int>& a, pair<int,int>& b){
    return a.second > b.second;       // smaller .second = higher priority
};
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);
```

### Custom comparator with a struct
```cpp
struct Task { int priority, id; };
struct Compare {
    bool operator()(const Task& a, const Task& b) {
        return a.priority < b.priority;   // max-heap by priority
    }
};
priority_queue<Task, vector<Task>, Compare> pq;
```

---

## DSA patterns

### 1. Dijkstra's shortest path
```cpp
vector<int> dijkstra(int src, vector<vector<pair<int,int>>>& adj) {
    int n = adj.size();
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>,
                   greater<pair<int,int>>> pq;   // {dist, node} min-heap
    dist[src] = 0;
    pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;               // stale entry, skip
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

### 2. Kth largest element — min-heap of size k
```cpp
int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> mn;  // min-heap
    for (int x : nums) {
        mn.push(x);
        if (mn.size() > k) mn.pop();   // keep only k largest; top = kth largest
    }
    return mn.top();
}
```
> Trick: to track the **K largest**, use a **min**-heap of size K (pop the smallest). For **K smallest**, use a max-heap of size K.

### 3. Merge K sorted lists
```cpp
// push the head of each list; pop the smallest, push its next
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
// {value, listIndex} ... pop min, advance that list, push next value
```

### 4. Median from data stream (two heaps)
```cpp
priority_queue<int> lo;                                   // max-heap (lower half)
priority_queue<int, vector<int>, greater<int>> hi;        // min-heap (upper half)
// keep sizes balanced; median = lo.top() or average of both tops
```

### 5. Task Scheduler / merge intervals by end time
Use a heap keyed on the field you always need the extreme of.

---

## priority_queue vs set
Both give you ordered access. Differences:
| | priority_queue | set/multiset |
|---|---|---|
| Get min/max | O(1) top | O(1) begin/rbegin |
| Insert | O(log n) | O(log n) |
| Remove arbitrary element | ❌ only top | ✅ O(log n) by value |
| Search arbitrary | ❌ | ✅ O(log n) |
| Duplicates | ✅ | multiset only |

Use a `multiset` instead of a heap when you need to **remove arbitrary elements** (e.g., sliding window median).

## Gotchas
- ⚠️ `pop()` returns void — read `top()` first.
- ⚠️ `top()` returns a **const reference** — you cannot modify the top in place. To "update" a value, push a new entry and skip stale ones (the `if (d > dist[u]) continue;` trick).
- ⚠️ No way to remove a non-top element — if you need that, use `multiset`.
- ⚠️ Easy to get min/max backwards — remember: `greater` = min-heap.
