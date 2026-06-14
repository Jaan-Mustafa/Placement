# queue — FIFO

`#include <queue>`

First-In-First-Out. A **container adaptor** over `deque` by default. Insert at the back, remove from the front. No iteration or indexing.

---

## Construction
```cpp
queue<int> q;                       // default (deque-backed)
queue<pair<int,int>> qp;            // queue of pairs (grid BFS)

deque<int> d = {1,2,3};
queue<int> q2(d);                   // front = 1
```

---

## Full function list
| Function | Description | Complexity |
|---|---|---|
| `q.push(x)` | Enqueue at back | O(1) |
| `q.emplace(args)` | Construct in place at back | O(1) |
| `q.pop()` | Dequeue from front — **returns void** | O(1) |
| `q.front()` | Reference to front (next out) | O(1) |
| `q.back()` | Reference to back (last in) | O(1) |
| `q.size()` | Number of elements | O(1) |
| `q.empty()` | True if empty | O(1) |
| `q.swap(q2)` | Swap two queues | O(1) |

> No `clear()`, no iterators. Drain in a loop to empty.

---

## Examples

### Basics
```cpp
queue<int> q;
q.push(1); q.push(2); q.push(3);   // front [1,2,3] back
cout << q.front();                 // 1
cout << q.back();                  // 3
q.pop();                           // removes 1, front -> 2
cout << q.size();                  // 2

while (!q.empty()) {               // drain (prints 2 3)
    cout << q.front() << " ";
    q.pop();
}
```

> ⚠️ Like stack, `pop()` returns nothing. Read `front()` first, then `pop()`.

---

## DSA patterns

### 1. BFS on a graph (shortest path in unweighted graph)
```cpp
vector<int> bfs(int start, vector<vector<int>>& adj) {
    int n = adj.size();
    vector<int> dist(n, -1);
    queue<int> q;
    q.push(start);
    dist[start] = 0;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int nbr : adj[node]) {
            if (dist[nbr] == -1) {           // not visited
                dist[nbr] = dist[node] + 1;
                q.push(nbr);
            }
        }
    }
    return dist;
}
```

### 2. BFS on a grid (4-directional)
```cpp
int dr[] = {-1, 1, 0, 0};
int dc[] = {0, 0, -1, 1};

int shortestPath(vector<vector<int>>& grid) {
    int n = grid.size(), m = grid[0].size();
    vector<vector<bool>> vis(n, vector<bool>(m, false));
    queue<pair<int,int>> q;
    q.push({0, 0});
    vis[0][0] = true;
    int steps = 0;
    while (!q.empty()) {
        int sz = q.size();
        for (int k = 0; k < sz; k++) {       // process whole level
            auto [r, c] = q.front(); q.pop();
            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m
                    && !vis[nr][nc] && grid[nr][nc] == 0) {
                    vis[nr][nc] = true;
                    q.push({nr, nc});
                }
            }
        }
        steps++;
    }
    return steps;
}
```

### 3. Level-order traversal of a binary tree
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();                   // nodes on this level
        vector<int> level;
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

> The `int sz = q.size()` snapshot before the inner loop is the standard trick to process **one level at a time**.

---

## queue vs deque vs priority_queue
| Need | Use |
|---|---|
| Plain FIFO, BFS | `queue` |
| Insert/remove at **both** ends | `deque` |
| Always pull min/max next | `priority_queue` |

## When to use a queue
- BFS (graphs, grids, trees) — shortest path in unweighted graphs.
- Level-order / breadth-first processing.
- Producer–consumer / task buffering, sliding-window counts.

## Gotchas
- ⚠️ `front()`/`pop()` on empty queue is UB — check `empty()`.
- ⚠️ No random access — if you need to peek inside, use a `deque` directly.
- For level-by-level BFS, snapshot `q.size()` **before** the inner loop (it changes as you push children).
