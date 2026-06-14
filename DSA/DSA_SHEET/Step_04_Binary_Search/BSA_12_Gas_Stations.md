# Minimize Max Distance to Gas Station  🔴

**Reference:** [LeetCode 774 — Minimize Max Distance to Gas Station](https://leetcode.com/problems/minimize-max-distance-to-gas-station/) (premium) · [GeeksforGeeks](https://www.geeksforgeeks.org/minimize-max-distance-gas-station/)

---

## Problem
Given sorted positions of existing gas stations, add `k` **new** stations anywhere to minimize the maximum distance between adjacent stations. Return that minimized max distance (a real number).

```
Input:  stations = [1,2,3,4,5,6,7,8,9,10], k = 9    Output: 0.50000
```

---

## Intuition: binary search on the real-valued answer
The answer (a distance `d`) lies in `[0, maxGap]`. For a candidate `d`, the number of stations needed in a gap of length `g` is `floor(g / d)`. Sum these across all gaps; if total ≤ k, `d` is feasible → try smaller. Since `d` is real, run a fixed number of iterations (or until the range is tiny).

---

## Code (C++)
```cpp
int stationsNeeded(vector<int>& st, double d) {
    int count = 0;
    for (int i = 1; i < st.size(); i++) {
        double gap = st[i] - st[i - 1];
        count += (int)(gap / d);            // floor; if exactly divisible, subtract 1 (edge)
        if (gap / d == (int)(gap / d)) count--;
    }
    return count;
}

double minMaxDist(vector<int>& st, int k) {
    double low = 0, high = 0;
    for (int i = 1; i < st.size(); i++) high = max(high, (double)(st[i] - st[i-1]));

    for (int iter = 0; iter < 100; iter++) {     // ~100 iters → high precision
        double mid = (low + high) / 2.0;
        if (stationsNeeded(st, mid) <= k) high = mid;   // feasible; shrink
        else low = mid;
    }
    return high;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · iterations) — iterations fixed (~100) for precision |
| Space | O(1) |

> Real-valued binary search: instead of `low <= high`, iterate a fixed count (or until `high - low < 1e-6`). Heap-based greedy also solves this but is messier; BS-on-answer is cleanest.
