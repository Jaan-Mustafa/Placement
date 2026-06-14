# Minimum Multiplications to Reach End  🟡

**Reference:** [GeeksforGeeks — Minimum Multiplications to Reach End](https://www.geeksforgeeks.org/minimum-multiplications-reach-end-given-number/)

---

## Problem
Given an array `arr`, a `start` number and an `end` number, repeatedly multiply the current value by any element of `arr` taking modulo `100000`. Find the minimum number of multiplications to transform `start` into `end`, or `-1` if impossible.

```
Input: arr=[2,5,7], start=3, end=30
Output: 2   (3*2=6, 6*5=30)
```

---

## Intuition
Treat each number in `0..99999` as a graph node. From a number `x`, multiplying by `arr[i]` (mod 100000) gives an edge to a neighbour. Every multiplication costs `1`, so the graph is unweighted and **BFS** from `start` gives the minimum number of steps. We track `steps[]` for each of the 100000 states and stop the moment we dequeue `end`.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int minimumMultiplications(vector<int>& arr, int start, int end) {
    const int MOD = 100000;
    if (start == end) return 0;

    vector<int> steps(MOD, -1);          // -1 == unvisited
    queue<int> q;
    steps[start] = 0;
    q.push(start);

    while (!q.empty()) {
        int x = q.front(); q.pop();
        for (int a : arr) {
            int nxt = (x * a) % MOD;      // next state after multiplication
            if (steps[nxt] == -1) {       // first visit => shortest
                steps[nxt] = steps[x] + 1;
                if (nxt == end) return steps[nxt];
                q.push(nxt);
            }
        }
    }
    return -1;                            // end unreachable
}
```

---

## Complexity
| | |
|---|---|
| Time | O(100000 · |arr|) |
| Space | O(100000) |

> The state space is bounded by the modulus (100000), which is what keeps this BFS finite even though multiplication could otherwise grow without limit.
