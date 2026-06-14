# Pascal's Triangle  🔴

**Reference:** [LeetCode 118 — Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/) · [LeetCode 119 — Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii/)

---

## Problem
Three common variants:
1. Given `(r, c)`, find the element at row `r`, column `c` (1-indexed).
2. Print the `r`-th row.
3. Generate the entire triangle up to `n` rows.

```
Row 1:        1
Row 2:       1 1
Row 3:      1 2 1
Row 4:     1 3 3 1
Row 5:    1 4 6 4 1
```

---

## Intuition
The element at row `r`, column `c` is the binomial coefficient `C(r-1, c-1)`. A whole row `r` can be generated in O(r) by iteratively multiplying: start at 1 and use `ans = ans * (r - i) / i`. This avoids computing factorials (overflow-prone).

---

## Code (C++)
```cpp
// Variant 2: generate the r-th row in O(r)
vector<int> generateRow(int r) {
    long long ans = 1;
    vector<int> row;
    row.push_back(1);
    for (int c = 1; c < r; c++) {
        ans = ans * (r - c);    // C(r-1, c) from C(r-1, c-1)
        ans = ans / c;
        row.push_back((int)ans);
    }
    return row;
}

// Variant 3: full triangle
vector<vector<int>> generate(int n) {
    vector<vector<int>> tri;
    for (int r = 1; r <= n; r++)
        tri.push_back(generateRow(r));
    return tri;
}
```

---

## Complexity
| Variant | Time | Space |
|---|---|---|
| Single element C(r,c) | O(c) | O(1) |
| One row | O(r) | O(r) |
| Full triangle | O(n²) | O(n²) |

> Building each row by summing adjacent elements of the previous row is also O(n²) but needs the previous row stored; the multiplicative formula generates any row independently.
