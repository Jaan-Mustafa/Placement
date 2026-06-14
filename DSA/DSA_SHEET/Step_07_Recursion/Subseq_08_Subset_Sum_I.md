# Subset Sum I (All Subset Sums)  🟡

**Reference:** [GeeksforGeeks — Subset sums](https://www.geeksforgeeks.org/sum-of-all-subsets-set-sum/) · [Coding Ninjas — Subset Sum]

---

## Problem
Return the sums of **all** subsets of an array, in any order.

```
Input:  [2, 3]
Output: [0, 2, 3, 5]   (sums of {}, {2}, {3}, {2,3})
```

---

## Intuition
The classic pick/not-pick tree, but instead of building subsets we carry a running `sum`. At the leaf (end of array), record the accumulated sum. There are 2ⁿ leaves → 2ⁿ sums.

---

## Code (C++)
```cpp
void solve(int i, vector<int>& a, int sum, vector<int>& res) {
    if (i == a.size()) { res.push_back(sum); return; }
    solve(i + 1, a, sum + a[i], res);     // include a[i]
    solve(i + 1, a, sum, res);            // exclude a[i]
}

vector<int> subsetSums(vector<int>& a) {
    vector<int> res;
    solve(0, a, 0, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2ⁿ) |
| Space | O(n) recursion + O(2ⁿ) output |

> No backtracking cleanup needed here because `sum` is passed by value — each branch gets its own copy.
