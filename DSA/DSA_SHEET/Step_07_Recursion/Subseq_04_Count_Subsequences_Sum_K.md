# Count All Subsequences with Sum K  🟡

**Reference:** [GeeksforGeeks — Count subsequences with sum K](https://www.geeksforgeeks.org/perfect-sum-problem/)

---

## Problem
Count how many subsequences of an array sum to exactly K.

```
Input:  [1, 2, 1], K = 2    Output: 2   ([1,1] and [2])
```

---

## Intuition
The pick/not-pick tree again, but instead of collecting subsets we **count** the ones that reach the target. At each index, recurse twice (include → subtract from target, exclude). Base case: when the array ends, return 1 if the remaining target is 0, else 0.

---

## Code (C++)
```cpp
int countSubseq(int i, vector<int>& a, int target) {
    if (target == 0) return 1;            // found a valid subsequence
    if (i == a.size()) return 0;          // ran out without hitting target

    int pick = 0;
    if (a[i] <= target)
        pick = countSubseq(i + 1, a, target - a[i]);   // include a[i]
    int notPick = countSubseq(i + 1, a, target);       // exclude a[i]
    return pick + notPick;
}
```

> ⚠️ The `target == 0` early-return works for **non-negative** values. With possible zeros/negatives, only check the target at `i == a.size()`.

---

## Complexity
| | |
|---|---|
| Time | O(2ⁿ) pure recursion |
| Space | O(n) recursion |

> This is the recursion that, when **memoized** on `(i, target)`, becomes the DP "count subsets with sum K" (Step 16).
