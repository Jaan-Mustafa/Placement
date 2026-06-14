# Minimum Cost to Cut a Stick  🔴

**Reference:** https://leetcode.com/problems/minimum-cost-to-cut-a-stick/

---

## Problem

A stick of length `n` is given along with an array `cuts` listing positions where it must be cut. The cost of a single cut equals the current length of the stick being cut. You may perform the cuts in any order. Return the minimum total cost.

```
n = 7, cuts = [1, 3, 4, 5]
One optimal order gives total cost = 16
```

---

## Intuition

This is **partition DP on intervals defined by cut positions**. Add the two endpoints `0` and `n` to `cuts`, then **sort**. Now consider any open segment bounded by two chosen markers `cuts[i]` and `cuts[j]` (with no committed cut strictly between them yet); we decide which cut `k` (strictly between `i` and `j`) to make *first* in that segment.

Define `f(i, j)` = min cost to perform all cuts strictly between markers `i` and `j`.

Recurrence (partition over the first cut `k` made inside `(i, j)`):
- `f(i, j) = min over k in (i, j) of ( cuts[j] - cuts[i] + f(i, k) + f(k, j) )`
- `cuts[j] - cuts[i]` is the length of the current piece (the cost of whichever cut we make first here).

Base case: if there is no cut between `i` and `j` (`j - i == 1`), `f(i, j) = 0`.

Sorting + adding endpoints makes the sub-problems independent intervals — that is what unlocks the DP.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

int minCost(int n, vector<int>& cuts) {
    cuts.push_back(0);
    cuts.push_back(n);
    sort(cuts.begin(), cuts.end());
    int m = cuts.size();                       // markers including 0 and n

    // dp[i][j]: min cost to cut everything strictly between marker i and j
    vector<vector<int>> dp(m, vector<int>(m, 0));

    // increasing interval length so sub-intervals are ready
    for (int len = 2; len < m; ++len) {
        for (int i = 0; i + len < m; ++i) {
            int j = i + len;
            int best = INT_MAX;
            for (int k = i + 1; k < j; ++k) {  // first cut to make = k
                int cost = (cuts[j] - cuts[i]) // current piece length
                         + dp[i][k] + dp[k][j];
                best = min(best, cost);
            }
            dp[i][j] = best;
        }
    }
    return dp[0][m - 1];
}
```

> Memoized recursion works too, but tabulation by interval length is clean and avoids stack depth. Space cannot be trivially reduced below O(m²) because cells depend on arbitrary sub-intervals.

---

## Complexity

| | |
|---|---|
| Time | O(m³) where m = cuts.size()+2 |
| Space | O(m²) |

> ⚠️ You MUST sort `cuts` and prepend/append `0` and `n` first. The cost term is `cuts[j] - cuts[i]` (boundary positions), not `cuts[k]`. Initialize length-1 intervals to 0.
