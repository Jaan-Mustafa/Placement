# Aggressive Cows  🔴

**Reference:** [SPOJ AGGRCOW](https://www.spoj.com/problems/AGGRCOW/) · [GeeksforGeeks](https://www.geeksforgeeks.org/aggressive-cows-detailed-solution-using-binary-search/)

---

## Problem
Place `k` cows in `n` stalls (positions given) so that the **minimum distance** between any two cows is **as large as possible**. Return that largest minimum distance.

```
Input:  stalls = [0, 3, 4, 7, 10, 9], k = 4    Output: 3
```

---

## Intuition: binary search on the answer (distance)
Sort the stalls. The answer (a distance) ranges from 1 to `maxPos - minPos`. A larger minimum distance is **harder** to satisfy (fewer cows fit) — monotonic. Binary search for the **largest** distance for which we can still place all `k` cows.

Feasibility (greedy): place the first cow at stall 0, then place each next cow at the first stall ≥ `lastPlaced + dist`.

---

## Code (C++)
```cpp
bool canPlace(vector<int>& stalls, int k, int dist) {
    int count = 1, last = stalls[0];
    for (int i = 1; i < stalls.size(); i++) {
        if (stalls[i] - last >= dist) {     // far enough → place a cow
            count++;
            last = stalls[i];
        }
        if (count >= k) return true;
    }
    return false;
}

int aggressiveCows(vector<int>& stalls, int k) {
    sort(stalls.begin(), stalls.end());
    int low = 1, high = stalls.back() - stalls.front();
    int ans = 0;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (canPlace(stalls, k, mid)) {
            ans = mid;          // feasible; try a larger min-distance
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) sort + O(N · log(maxDist)) search |
| Space | O(1) |

> Note the direction flip: here we **maximize** the answer, so on success we go **right** (`low = mid + 1`). Compare with Koko, where we minimized and went left.
