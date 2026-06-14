# Fibonacci Number (Recursion)  🟢

**Reference:** [LeetCode 509 — Fibonacci Number](https://leetcode.com/problems/fibonacci-number/) · [GeeksforGeeks](https://www.geeksforgeeks.org/program-for-nth-fibonacci-number/)

---

## Problem
Return the Nth Fibonacci number, where `F(0)=0`, `F(1)=1`, and `F(n)=F(n-1)+F(n-2)`.

```
Input:  N = 5    Output: 5     (0,1,1,2,3,5)
Input:  N = 0    Output: 0
```

---

## Intuition
Direct translation of the recurrence — but plain recursion recomputes the same values exponentially. For learning the recurrence it's fine; in practice use memoization or an iterative O(N) loop.

---

## Code (C++)
```cpp
// Pure recursion — O(2^N), exponential (this is the "basics" version)
int fib(int n) {
    if (n <= 1) return n;            // base cases F(0)=0, F(1)=1
    return fib(n - 1) + fib(n - 2);
}

// Iterative O(N) time, O(1) space (preferred in practice)
int fibIterative(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

---

## Why pure recursion is slow
`fib(5)` calls `fib(4)` and `fib(3)`; `fib(4)` again calls `fib(3)`... the same subproblems repeat → **O(2ⁿ)**. Memoizing (cache results) brings it to O(N) — this is the gateway to Dynamic Programming (Step 16).

## Complexity
| Version | Time | Space |
|---|---|---|
| Pure recursion | O(2ⁿ) | O(N) stack |
| Iterative | O(N) | O(1) |
