# Frog Jump with K Distances  🟡

**Reference:** https://www.geeksforgeeks.org/minimal-cost-to-make-an-array-good-by-jumps-of-at-most-k/ , https://www.codingninjas.com/studio/problems/minimal-cost_8180930

---

## Problem

Same as the Frog Jump problem, but now from stone `i` the frog can jump to any stone in the range `i+1, i+2, ..., i+k`. The cost of jumping from `i` to `j` is `abs(height[i] - height[j])`. Find the **minimum total cost** to reach the last stone from stone `0`.

```
Input : height = [10, 30, 40, 50, 20], k = 3
Output: 30
Explanation:
  0 -> 1 -> 4 : |10-30| + |30-20| = 20 + 10 = 30
  0 -> 4 directly is not allowed (distance 4 > k=3)
Minimum = 30
```

---

## Intuition

Generalize the 1/2-step recurrence to up to `k` steps. To reach stone `i`, try every possible previous stone `i-j` for `j` in `1..k` (as long as `i-j >= 0`), take the minimum.

**DP state:** `dp[i]` = minimum cost to reach stone `i` from stone `0`.
**Recurrence:**
```
dp[i] = min over j in [1..k], (i-j) >= 0 of
            dp[i-j] + abs(height[i] - height[i-j])
```
**Base case:** `dp[0] = 0`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int frogJumpK(vector<int>& height, int k) {
    int n = height.size();
    if (n == 1) return 0;
    vector<int> dp(n, 0);
    dp[0] = 0;                               // base: starting stone
    for (int i = 1; i < n; ++i) {
        int best = INT_MAX;
        for (int j = 1; j <= k; ++j) {       // try every jump length 1..k
            if (i - j >= 0) {
                int cost = dp[i - j] + abs(height[i] - height[i - j]);
                best = min(best, cost);
            }
        }
        dp[i] = best;
    }
    return dp[n - 1];
}

int main() {
    vector<int> h = {10, 30, 40, 50, 20};
    cout << frogJumpK(h, 3) << "\n"; // 30
    return 0;
}
```

> note: Unlike the 2-step variant, this generally needs the full `dp[]` array because `dp[i]` may depend on any of the previous `k` states. Space cannot trivially drop below `O(n)` here (a deque/sliding-window optimization can help the time factor but not the array storage).

---

## Complexity

| | |
|---|---|
| Time | O(n * k) |
| Space | O(n) |

> ⚠️ Always bound the inner loop with `i - j >= 0` so you don't read negative indices for the early stones.
> ⚠️ With `k = 2` this reduces to the standard Frog Jump problem — a good sanity check.
