# Candy  🔴

**Reference:** [LeetCode 135 — Candy](https://leetcode.com/problems/candy/)

---

## Problem
`n` children stand in a line with `ratings[i]`. Each child gets at least one candy, and a child with a higher rating than an adjacent neighbor must get more candies than that neighbor. Return the minimum total candies.

```
Input:  [1,0,2]   Output: 5   (2,1,2)
Input:  [1,2,2]   Output: 4   (1,2,1)
```

---

## Intuition
Each child's candy is constrained by both neighbors. Handle the two directions separately with **two greedy passes**:
1. **Left → right:** if `ratings[i] > ratings[i-1]`, give one more than the left neighbor.
2. **Right → left:** if `ratings[i] > ratings[i+1]`, ensure at least one more than the right neighbor (take the `max` so the left-pass result isn't clobbered).

Taking the max of the two passes satisfies both inequalities with the minimum candy.

---

## Code (C++)
```cpp
int candy(vector<int>& ratings) {
    int n = ratings.size();
    vector<int> candies(n, 1);         // everyone starts with 1

    // left to right: satisfy left neighbor constraint
    for (int i = 1; i < n; i++)
        if (ratings[i] > ratings[i-1])
            candies[i] = candies[i-1] + 1;

    // right to left: satisfy right neighbor constraint, keep the larger value
    for (int i = n - 2; i >= 0; i--)
        if (ratings[i] > ratings[i+1])
            candies[i] = max(candies[i], candies[i+1] + 1);

    int total = 0;
    for (int c : candies) total += c;
    return total;
}
```

---

## Dry run
`[1,0,2]`: start [1,1,1]. L→R: [1,1,2]. R→L: i=1 (0<2 no), i=0 (1>0) → candies[0]=max(1,2)=2 → [2,1,2]. Total **5**.

## Complexity
| | |
|---|---|
| Time | O(N) (two passes) |
| Space | O(N) for the candy array |

> ⚠️ In the right-to-left pass you MUST use `max(...)` — overwriting directly would destroy a larger value already assigned by the left pass. A one-pass O(1)-space version exists using up/down slope counting but is trickier.
