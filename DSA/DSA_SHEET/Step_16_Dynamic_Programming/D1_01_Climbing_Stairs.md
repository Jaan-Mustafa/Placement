# Climbing Stairs  🟢

**Reference:** https://leetcode.com/problems/climbing-stairs/

---

## Problem

You are climbing a staircase with `n` steps. Each time you can climb either **1** or **2** steps. In how many distinct ways can you reach the top?

```
Input : n = 3
Output: 3
Explanation: The ways are:
  1 + 1 + 1
  1 + 2
  2 + 1
```

---

## Intuition

To reach step `i`, the last move was either a single step (from `i-1`) or a double step (from `i-2`). So the number of ways to reach `i` is the sum of the ways to reach `i-1` and `i-2`. This is exactly the Fibonacci recurrence.

**DP state:** `dp[i]` = number of distinct ways to reach step `i`.
**Recurrence:** `dp[i] = dp[i-1] + dp[i-2]`.
**Base cases:** `dp[0] = 1` (one way to "stand" at the ground), `dp[1] = 1`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int climbStairs(int n) {
    if (n <= 1) return 1;
    vector<int> dp(n + 1, 0);
    dp[0] = 1;                     // base: stay at ground (empty path)
    dp[1] = 1;                     // base: single way to reach step 1
    for (int i = 2; i <= n; ++i)
        dp[i] = dp[i - 1] + dp[i - 2]; // come from i-1 or i-2
    return dp[n];
}

int main() {
    cout << climbStairs(3) << "\n"; // 3
    cout << climbStairs(5) << "\n"; // 8
    return 0;
}
```

> note: Since `dp[i]` depends only on the previous two values, this can be reduced to `O(1)` space using two variables (`prev1`, `prev2`), just like space-optimized Fibonacci.

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(n) (reducible to O(1) with two variables) |

> ⚠️ The answer grows like Fibonacci; for large `n` use `long long` to avoid overflow (LeetCode constraints keep `n <= 45`, which fits in `int`).
