# Koko Eating Bananas  🟡

**Reference:** [LeetCode 875 — Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)

---

## Problem
Koko has `piles` of bananas and `h` hours. At speed `k` bananas/hour she eats `ceil(pile/k)` hours per pile. Find the **minimum** `k` so she finishes all piles within `h` hours.

```
Input:  piles = [3, 6, 7, 11], h = 8    Output: 4
Input:  piles = [30, 11, 23, 4, 20], h = 5    Output: 30
```

---

## Intuition: binary search on the eating speed
Speed ranges from 1 to `max(piles)`. Hours needed is **monotonic**: higher speed → fewer hours. We binary search for the smallest speed whose total hours ≤ h. If `mid` is feasible, try smaller; else go higher.

---

## Code (C++)
```cpp
long long hoursNeeded(vector<int>& piles, int speed) {
    long long hours = 0;
    for (int p : piles)
        hours += (p + speed - 1) / speed;       // ceil(p / speed)
    return hours;
}

int minEatingSpeed(vector<int>& piles, int h) {
    int low = 1, high = *max_element(piles.begin(), piles.end());
    int ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (hoursNeeded(piles, mid) <= h) {
            ans = mid;              // feasible; try slower
            high = mid - 1;
        } else {
            low = mid + 1;          // too slow; go faster
        }
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log(max pile)) |
| Space | O(1) |

> The signature of a "BS on answer" problem: a **monotonic feasibility** function (here, hours decrease as speed increases). Identify the search range and the check.
