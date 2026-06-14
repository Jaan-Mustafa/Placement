# Why Priority Queue in Dijkstra  🟡

**Reference:** [GeeksforGeeks — Dijkstra's Shortest Path Algorithm](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/)

---

## Problem
This is a conceptual page: **why** does Dijkstra use a min-heap / priority queue rather than a plain FIFO queue like BFS?

```
Question: BFS uses a queue and works for unit weights.
          Why can't we just use a queue for weighted Dijkstra?
```

---

## Intuition
Dijkstra is **greedy**: at each step it must finalise the *currently closest* unfinalised node, because for non-negative weights that node's distance can never improve later. A plain FIFO queue gives you nodes in insertion order, not in order of smallest distance, so you would finalise nodes prematurely and have to reprocess them many times.

A **min-heap / priority queue** always hands back the minimum-distance node in `O(log V)`, which is exactly the greedy choice Dijkstra needs. This:
- **Avoids reprocessing** — a popped node already has its final distance (we only skip stale duplicates).
- **Scales with sparse graphs** — `O(E log V)` instead of the `O(V^2)` you'd pay scanning a flat array for the minimum on dense or even sparse graphs.

The simple array version (linear-scan for the minimum) is fine and even optimal for **dense** graphs (`E ≈ V^2`), but on sparse graphs the heap wins decisively.

---

## Code (C++)
```cpp
// The core heap pattern: always expand the closest unfinalised node.
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // {dist,node}
pq.push({0, src});
while (!pq.empty()) {
    auto [d, node] = pq.top(); pq.pop();   // O(log V) min extraction
    if (d > dist[node]) continue;          // stale entry => skip, no reprocessing
    for (auto& [nbr, w] : adj[node])
        if (d + w < dist[nbr]) {
            dist[nbr] = d + w;
            pq.push({dist[nbr], nbr});      // re-prioritise neighbour
        }
}
```

---

## Complexity
| Approach | Time | Best for |
|---|---|---|
| Flat array (linear-scan min) | O(V²) | dense graphs (E ≈ V²) |
| Priority queue / min-heap | O(E log V) | sparse graphs (E ≪ V²) |
| Fibonacci heap (theoretical) | O(E + V log V) | rarely used in practice |

> A plain FIFO queue would finalise nodes in the wrong order and force endless reprocessing — the min-heap is what makes Dijkstra's greedy choice both correct and efficient.
