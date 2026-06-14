# Introduction to DP (memoization vs tabulation)  🟢

**Reference:** https://leetcode.com/problems/fibonacci-number/ , https://www.geeksforgeeks.org/dynamic-programming/

---

## Problem

Dynamic Programming (DP) is an optimization technique over plain recursion. Whenever a recursive solution has **overlapping subproblems** (the same subproblem is solved many times) and **optimal substructure** (the answer to a problem can be built from answers to smaller subproblems), we can store (cache) the results of subproblems and reuse them.

We use the classic **Fibonacci** sequence as a running example:

```
fib(0) = 0
fib(1) = 1
fib(n) = fib(n-1) + fib(n-2)

n   : 0 1 2 3 4 5 6  7  8 ...
fib : 0 1 1 2 3 5 8 13 21 ...
```

A naive recursion for `fib(n)` recomputes the same values exponentially many times — `O(2^n)`. DP brings this down to `O(n)`.

---

## Intuition

There are three standard ways to write a DP solution:

1. **Memoization (Top-Down):** Write the natural recursion, but cache each computed answer in a `dp[]` array (initialized to a sentinel like `-1`). Before computing `fib(n)`, check if it is already cached; if so, return it. We start from the top (`n`) and recurse down to base cases.

2. **Tabulation (Bottom-Up):** Remove recursion entirely. Define `dp[i]` = answer for state `i`, fill base cases first, then iterate `i` from small to large using the recurrence. We build *up* from base cases to the final answer.

3. **Space Optimization:** In tabulation, if `dp[i]` only depends on the last few states (here `dp[i-1]` and `dp[i-2]`), we don't need the whole array — just a couple of variables. This reduces space from `O(n)` to `O(1)`.

**DP state:** `dp[i]` = the i-th Fibonacci number.
**Recurrence:** `dp[i] = dp[i-1] + dp[i-2]`, with base cases `dp[0] = 0`, `dp[1] = 1`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

// 1) MEMOIZATION (Top-Down): recursion + cache
int fibMemo(int n, vector<int>& dp) {
    if (n <= 1) return n;          // base cases
    if (dp[n] != -1) return dp[n]; // already computed -> reuse
    // store result before returning
    return dp[n] = fibMemo(n - 1, dp) + fibMemo(n - 2, dp);
}

// 2) TABULATION (Bottom-Up): iterative, fill dp[] from base cases up
int fibTab(int n) {
    if (n <= 1) return n;
    vector<int> dp(n + 1, 0);
    dp[0] = 0;                     // base case
    dp[1] = 1;                     // base case
    for (int i = 2; i <= n; ++i)   // build up using recurrence
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}

// 3) SPACE OPTIMIZED: only keep the last two values -> O(1) space
int fibOpt(int n) {
    if (n <= 1) return n;
    int prev2 = 0;                 // fib(i-2)
    int prev1 = 1;                 // fib(i-1)
    for (int i = 2; i <= n; ++i) {
        int cur = prev1 + prev2;   // fib(i)
        prev2 = prev1;             // slide the window forward
        prev1 = cur;
    }
    return prev1;
}

int main() {
    int n = 8;
    vector<int> dp(n + 1, -1);     // -1 = "not computed yet"
    cout << "Memo : " << fibMemo(n, dp) << "\n"; // 21
    cout << "Tab  : " << fibTab(n)      << "\n"; // 21
    cout << "Opt  : " << fibOpt(n)      << "\n"; // 21
    return 0;
}
```

> note: Memoization uses `O(n)` recursion stack space; tabulation removes the stack; the space-optimized version uses only `O(1)` extra space. Prefer tabulation/space-optimization when recursion depth could cause stack overflow.

---

## Complexity

| | |
|---|---|
| Time | O(n) for all three variants (each state computed once) |
| Space | Memoization: O(n) array + O(n) stack; Tabulation: O(n); Space optimized: O(1) |

> ⚠️ Naive recursion **without** caching is `O(2^n)` — always confirm overlapping subproblems before assuming a recursive solution is efficient.
> ⚠️ Fibonacci grows fast; for large `n` use `long long` (or modular arithmetic) to avoid `int` overflow.
