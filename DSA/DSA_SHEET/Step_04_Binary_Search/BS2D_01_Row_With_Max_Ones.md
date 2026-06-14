# Row with Maximum 1's  🟡

**Reference:** [GeeksforGeeks — Row with max 1s](https://www.geeksforgeeks.org/find-the-row-with-maximum-number-1s/)

---

## Problem
Given a binary matrix where **each row is sorted** (0s then 1s), find the index of the row containing the most 1s. On ties, return the smallest index.

```
Input:  [[0,0,1,1],
         [0,1,1,1],
         [0,0,0,1],
         [0,0,0,0]]
Output: 1   (row 1 has three 1's)
```

---

## Intuition
Since each row is sorted, the count of 1s in a row = `cols - lowerBound(row, 1)` (index of the first 1). Compute this per row with binary search and track the maximum. This beats scanning every cell.

---

## Code (C++)
```cpp
int lowerBound(vector<int>& row, int x) {
    int lo = 0, hi = row.size() - 1, ans = row.size();
    while (lo <= hi) {
        int m = lo + (hi - lo) / 2;
        if (row[m] >= x) { ans = m; hi = m - 1; }
        else lo = m + 1;
    }
    return ans;
}

int rowWithMaxOnes(vector<vector<int>>& mat) {
    int n = mat.size(), m = mat[0].size();
    int bestRow = -1, maxOnes = 0;
    for (int i = 0; i < n; i++) {
        int ones = m - lowerBound(mat[i], 1);    // first '1' index → count
        if (ones > maxOnes) { maxOnes = ones; bestRow = i; }
    }
    return bestRow;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log M) |
| Space | O(1) |

> An even slicker O(N + M) approach: start at the top-right; move left on a 1 (count a new best row), move down on a 0.
