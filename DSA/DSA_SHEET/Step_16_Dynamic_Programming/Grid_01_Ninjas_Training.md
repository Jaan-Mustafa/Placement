# Ninja's Training  🟡

**Reference:** https://www.codingninjas.com/studio/problems/ninja-s-training_3621003

---

## Problem

A ninja trains over `n` days. Each day he can perform exactly one of 3 activities, and each activity gives some merit points (`points[day][task]`). The constraint is that he **cannot perform the same activity on two consecutive days**. Maximize the total merit points over all `n` days.

```
points =
  [[1, 2, 5],
   [3, 1, 1],
   [3, 3, 3]]

Day 0: pick task 2 -> 5
Day 1: pick task 0 -> 3  (can't repeat task 2)
Day 2: pick task 1 -> 3  (can't repeat task 0)
Answer = 5 + 3 + 3 = 11
```

---

## Intuition

State: `dp[day][last]` = max points obtainable from `day .. n-1` given that the activity chosen on the **previous** day was `last` (0,1,2 = an actual task, 3 = no previous task / "free").

Recurrence: on `day`, try every task `t != last`:

```
dp[day][last] = max over t!=last ( points[day][t] + dp[day+1][t] )
```

Bottom-up: process days from `0` upward, where `dp[day][last]` only depends on `day-1`. We carry the running best per "last task". The answer is the best value choosing any task on the final day with no restriction at the start (`last = 3`).

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int ninjaTraining(int n, vector<vector<int>>& points) {
    // dp[last] = best total from day 0..i, where 'last' is the task done on day i.
    // last in {0,1,2}; we use 4 columns so index 3 = "no previous task".
    vector<int> dp(4, 0);

    // Base case: day 0. If previous task is 'last', best today is the max
    // points among tasks != last.
    dp[0] = max(points[0][1], points[0][2]);
    dp[1] = max(points[0][0], points[0][2]);
    dp[2] = max(points[0][0], points[0][1]);
    dp[3] = max({points[0][0], points[0][1], points[0][2]});

    // Iterate the remaining days, building cur from previous day's dp.
    for (int day = 1; day < n; day++) {
        vector<int> cur(4, 0);
        for (int last = 0; last < 4; last++) {
            cur[last] = 0;
            for (int task = 0; task < 3; task++) {
                if (task != last) {                       // can't repeat yesterday's task
                    int val = points[day][task] + dp[task];
                    cur[last] = max(cur[last], val);
                }
            }
        }
        dp = cur;                                         // only previous day is needed
    }

    // No restriction before day 0 -> use 'last = 3'.
    return dp[3];
}

int main() {
    vector<vector<int>> points = {{1,2,5},{3,1,1},{3,3,3}};
    cout << ninjaTraining(points.size(), points) << "\n"; // 11
}
```

Space optimization: already applied — only two rows of size 4 (`dp` and `cur`) are kept instead of the full `n x 4` table, reducing space from O(n) rows to O(1).

---

## Complexity

| | |
|---|---|
| Time | O(n × 4 × 3) = O(n) |
| Space | O(1) (only two rows of 4 ints) |

> ⚠️ The `last = 3` ("no previous task") column is essential — it lets the very first day pick freely. Forgetting it (or initializing dp to garbage) leads to a wrong base case.
