# Course Schedule I  🟡

**Reference:** [LeetCode 207 — Course Schedule](https://leetcode.com/problems/course-schedule/)

---

## Problem
There are `numCourses` courses labeled `0..numCourses-1`. `prerequisites[i] = [a, b]` means you must take course `b` before course `a`. Return `true` if you can finish all courses.

```
Input:  numCourses = 2, prerequisites = [[1,0]]
Output: true    (take 0, then 1)

Input:  numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false   (0 and 1 depend on each other)
```

---

## Intuition
Model courses as nodes and a prerequisite `[a, b]` as a directed edge `b -> a` ("b before a"). Finishing all courses is possible if and only if this dependency graph is a DAG — a cycle means a set of courses mutually require each other and can never all be taken. Run Kahn's algorithm: if every course gets processed (count reaches `numCourses`), there is no cycle and all courses can be finished.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
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

        int done = 0;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            done++;                         // this course can be taken now
            for (int next : adj[node])
                if (--indegree[next] == 0)
                    q.push(next);
        }
        return done == numCourses;          // all courses finished => no cycle
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(V + E) |
| Space | O(V + E) |

> Watch the edge direction: `[a, b]` means `b -> a`. Getting it backwards still detects cycles correctly but is conceptually wrong and breaks Course Schedule II's ordering.
