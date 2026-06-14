# Find the Repeating and Missing Number  🔴

**Reference:** [GeeksforGeeks — Repeating and Missing](https://www.geeksforgeeks.org/find-a-repeating-and-a-missing-number/) · [LeetCode 645 — Set Mismatch](https://leetcode.com/problems/set-mismatch/)

---

## Problem
An array holds numbers `1..N`, but one number `x` is repeated (taking another's place) and one number `y` is missing. Find both.

```
Input:  [3, 1, 2, 5, 3]    Output: repeating = 3, missing = 4
```

---

## Intuition: math with sums and squares
Let S = sum of array, S² = sum of squares.
- Expected sum `Sn = N(N+1)/2`, expected square sum `S2n = N(N+1)(2N+1)/6`.
- `x - y = S - Sn`           ... (difference of values)
- `x² - y² = S² - S2n`  →  `x + y = (S² - S2n) / (x - y)`

Solving the two linear equations gives `x` (repeating) and `y` (missing) in O(N), O(1).

---

## Code (C++)
```cpp
pair<int,int> findMissingRepeating(vector<int>& a) {
    long long n = a.size();
    long long S = 0, S2 = 0;
    for (int v : a) { S += v; S2 += 1LL * v * v; }

    long long Sn  = n * (n + 1) / 2;
    long long S2n = n * (n + 1) * (2 * n + 1) / 6;

    long long diff = S - Sn;            // x - y
    long long sum  = (S2 - S2n) / diff; // x + y

    long long x = (diff + sum) / 2;     // repeating
    long long y = x - diff;             // missing
    return {(int)x, (int)y};
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> An XOR-based method also exists (avoids overflow): XOR all elements with 1..N to get `x^y`, isolate a differing bit to split numbers into two groups, and XOR within each group.
