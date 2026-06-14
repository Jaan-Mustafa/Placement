# Check if a Subsequence with Sum K Exists  🟡

**Reference:** [GeeksforGeeks — Subset sum problem](https://www.geeksforgeeks.org/subset-sum-problem/)

---

## Problem
Return **true** if there exists at least one subsequence summing to K.

```
Input:  [1, 2, 3, 4], K = 6    Output: true   ([2,4] or [1,2,3])
Input:  [1, 2, 3], K = 7       Output: false
```

---

## Intuition
Same pick/not-pick tree as the counting version, but **short-circuit**: as soon as one branch returns true, stop exploring. Using `||` naturally does this in C++ (the second operand isn't evaluated if the first is true).

---

## Code (C++)
```cpp
bool subsetExists(int i, vector<int>& a, int target) {
    if (target == 0) return true;             // found it
    if (i == a.size()) return false;

    bool pick = false;
    if (a[i] <= target)
        pick = subsetExists(i + 1, a, target - a[i]);
    if (pick) return true;                    // short-circuit

    return subsetExists(i + 1, a, target);    // exclude a[i]
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2ⁿ) worst, but often much less due to early exit |
| Space | O(n) recursion |

> Memoize `(i, target)` → O(N·K) DP. The boolean version is the gateway to the **0/1 knapsack** family of problems.
