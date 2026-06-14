# Maximum Sum Combination  🔴

**Reference:** [GeeksforGeeks — K maximum sum combinations from two arrays](https://www.geeksforgeeks.org/k-maximum-sum-combinations-two-arrays/) · [InterviewBit — Maximum Sum Combinations](https://www.interviewbit.com/problems/maximum-sum-combinations/)

---

## Problem
Given two equal-length arrays `A` and `B`, a "combination sum" is `A[i] + B[j]`. Return the **top `k` largest** combination sums.

```
Input:  A = [3, 2], B = [1, 4], k = 2
Output: [7, 6]   (3+4=7, 2+4=6)
```

---

## Intuition
Sort both arrays ascending. The single largest sum is `A[n-1] + B[n-1]`. From any pair of indices `(i, j)`, the next-largest candidates are `(i-1, j)` and `(i, j-1)`. Use a **max-heap on the sum** to expand outward from the top corner, popping the largest and pushing its two neighbors, using a `visited` set to avoid re-adding an index pair.

---

## Code (C++)
```cpp
vector<int> maxSumCombinations(vector<int>& A, vector<int>& B, int k) {
    int n = A.size();
    sort(A.begin(), A.end());
    sort(B.begin(), B.end());

    // max-heap of {sum, i, j}; default tuple ordering compares sum first
    priority_queue<tuple<int,int,int>> mx;
    set<pair<int,int>> seen;

    mx.push({A[n-1] + B[n-1], n-1, n-1});     // largest possible sum
    seen.insert({n-1, n-1});

    vector<int> res;
    while ((int)res.size() < k && !mx.empty()) {
        auto [s, i, j] = mx.top(); mx.pop();
        res.push_back(s);
        if (i > 0 && !seen.count({i-1, j})) {  // neighbor down in A
            mx.push({A[i-1] + B[j], i-1, j});
            seen.insert({i-1, j});
        }
        if (j > 0 && !seen.count({i, j-1})) {  // neighbor down in B
            mx.push({A[i] + B[j-1], i, j-1});
            seen.insert({i, j-1});
        }
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N + k log k) |
| Space | O(k) for heap + visited set |

> ⚠️ Without the `seen` set the pair `(i-1, j-1)` could be pushed twice (reached via two paths), corrupting the result. The default `priority_queue<tuple<...>>` is a max-heap and compares the first tuple element (the sum) first — exactly what we want.
