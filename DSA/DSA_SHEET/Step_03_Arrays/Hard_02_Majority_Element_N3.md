# Majority Element II (> N/3)  🔴

**Reference:** [LeetCode 229 — Majority Element II](https://leetcode.com/problems/majority-element-ii/)

---

## Problem
Find all elements appearing more than ⌊N/3⌋ times. There can be **at most two** such elements.

```
Input:  [3, 2, 3]       Output: [3]
Input:  [1, 1, 1, 3, 3, 2, 2, 2]    Output: [1, 2]
```

---

## Intuition: extended Boyer-Moore Voting
Since more than N/3 means at most 2 candidates, we track **two** candidates and **two** counts.
- If the element equals a candidate → increment that count.
- Else if a count is 0 → adopt the element as that candidate.
- Else → decrement both counts.

A second pass verifies each candidate actually exceeds N/3 (voting alone doesn't guarantee it).

---

## Code (C++)
```cpp
vector<int> majorityElement(vector<int>& nums) {
    int c1 = INT_MIN, c2 = INT_MIN, cnt1 = 0, cnt2 = 0;
    for (int x : nums) {
        if (x == c1) cnt1++;
        else if (x == c2) cnt2++;
        else if (cnt1 == 0) { c1 = x; cnt1 = 1; }
        else if (cnt2 == 0) { c2 = x; cnt2 = 1; }
        else { cnt1--; cnt2--; }
    }
    // verify
    cnt1 = cnt2 = 0;
    for (int x : nums) { if (x == c1) cnt1++; else if (x == c2) cnt2++; }
    vector<int> res;
    if (cnt1 > nums.size() / 3) res.push_back(c1);
    if (cnt2 > nums.size() / 3) res.push_back(c2);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) (two passes) |
| Space | O(1) |

> Generalization: for "> N/k" there can be at most k-1 candidates; the same voting idea extends with k-1 counters.
