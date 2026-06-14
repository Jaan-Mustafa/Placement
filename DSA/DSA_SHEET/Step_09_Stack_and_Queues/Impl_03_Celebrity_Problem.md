# Celebrity Problem  🔴

**Reference:** [GeeksforGeeks — The Celebrity Problem](https://www.geeksforgeeks.org/the-celebrity-problem/) · [LeetCode 277](https://leetcode.com/problems/find-the-celebrity/)

---

## Problem
In a party of `n` people, a *celebrity* is known by everyone but knows nobody. Given `knows(a, b)` (does a know b?), find the celebrity's index, or `-1`.

```
M = [[0,1,0],
     [0,0,0],
     [0,1,0]]
Output: 1   (person 1 knows nobody, everyone knows 1)
```

---

## Intuition
Use a **stack** (or two pointers) to eliminate candidates. Push everyone, then repeatedly pop two: if `a knows b`, `a` can't be the celebrity (discard `a`, keep `b`); otherwise `b` can't be (keep `a`). One candidate survives. **Verify** it: a true celebrity knows nobody and is known by all. The elimination is the stack-based core.

---

## Code (C++)
```cpp
// knows(a, b) returns true if a knows b
int celebrity(vector<vector<int>>& M, int n) {
    stack<int> st;
    for (int i = 0; i < n; i++) st.push(i);

    while (st.size() > 1) {
        int a = st.top(); st.pop();
        int b = st.top(); st.pop();
        if (M[a][b] == 1) st.push(b);       // a knows b -> a is out, b stays
        else st.push(a);                    // a doesn't know b -> b is out
    }
    int cand = st.top();

    // verify the candidate
    for (int i = 0; i < n; i++) {
        if (i == cand) continue;
        if (M[cand][i] == 1 || M[i][cand] == 0) return -1;  // not a celebrity
    }
    return cand;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — N-1 eliminations + O(N) verify |
| Space | O(N) for the stack (O(1) with the two-pointer variant) |

> ⚠️ The elimination only narrows to a *candidate*; you must verify it (knows nobody, known by all) — skipping verification fails when no celebrity exists.
