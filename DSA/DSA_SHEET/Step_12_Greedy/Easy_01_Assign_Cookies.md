# Assign Cookies  🟢

**Reference:** [LeetCode 455 — Assign Cookies](https://leetcode.com/problems/assign-cookies/)

---

## Problem
Each child `i` has a greed factor `g[i]` (minimum cookie size that satisfies them). Each cookie `j` has size `s[j]`. A cookie can satisfy at most one child. Maximize the number of content children.

```
Input:  g = [1,2,3], s = [1,1]    Output: 1
Input:  g = [1,2],   s = [1,2,3]  Output: 2
```

---

## Intuition
**Sort both arrays.** Give the smallest available cookie that can satisfy the least greedy remaining child. Using a larger cookie on a low-greed child would waste it — a cookie that fits a greedy child also fits a less greedy one, so we never lose by feeding the smallest-fitting cookie to the smallest child first.

Use two pointers: advance the child pointer only when the current cookie satisfies them; always advance the cookie pointer.

---

## Code (C++)
```cpp
int findContentChildren(vector<int>& g, vector<int>& s) {
    sort(g.begin(), g.end());          // children by greed
    sort(s.begin(), s.end());          // cookies by size
    int child = 0, cookie = 0;
    while (child < (int)g.size() && cookie < (int)s.size()) {
        if (s[cookie] >= g[child])     // cookie satisfies this child
            child++;                   // child is content, move on
        cookie++;                      // this cookie is used up either way
    }
    return child;                      // number of content children
}
```

---

## Dry run
`g=[1,2,3], s=[1,1]`: cookie 1 satisfies child(greed 1) → child=1; cookie 1 can't satisfy child(greed 2) → cookies exhausted. Answer **1**.

## Complexity
| | |
|---|---|
| Time | O(N log N + M log M) (sorting) |
| Space | O(1) extra |

> ⚠️ A cookie is consumed whether or not it satisfies a child — always increment `cookie`. Only increment `child` on a successful match.
