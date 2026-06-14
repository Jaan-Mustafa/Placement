# Course Schedule II  🟡

**Reference:** [LeetCode 210 — Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

---

## Problem
There are `numCourses` courses labeled `0..numCourses-1`. `prerequisites[i] = [a, b]` means course `b` must be taken before course `a`. Return any valid order to finish all courses, or an empty array if impossible.

```
Input:  numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: [0,1,2,3]   (or [0,2,1,3])

Input:  numCourses = 2, prerequisites = [[0,1],[1,0]]
Output: []          (cycle)
```

---

## Intuition
This is exactly Course Schedule I but we also record the order in which courses become free. Build edges `b -> a` and run Kahn's algorithm, appending each processed node to the answer. That append order is a valid topological order. If a cycle exists, fewer than `numCourses` nodes are processed, so the recorded order is incomplete and we must return an empty array instead.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adj(numCourses);
        vector<int> indegree(numCourses, 0);
        for (auto& p : prerequisites) {     // p = [a, b] => edge b -> a
            adj[p[1]].push_back(p[0]);
            indegree[p[0]]++;
        }

        queue<int> q;
        for (int i = 0; i < numCourses; i++)
            if (indegree[i] == 0)
                q.push(i);

        vector<int> order;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            order.push_back(node);          // record course in topo order
            for (int next : adj[node])
                if (--indegree[next] == 0)
                    q.push(next);
        }

        if ((int)order.size() != numCourses)
            return {};                      // cycle => no valid ordering
        return order;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Return `{}` (empty), not a partial order, when a cycle is detected — the order vector at that point contains only the acyclic prefix.
