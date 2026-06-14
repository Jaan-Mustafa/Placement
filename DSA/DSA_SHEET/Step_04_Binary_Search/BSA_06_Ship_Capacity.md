# Capacity to Ship Packages Within D Days  🟡

**Reference:** [LeetCode 1011 — Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/)

---

## Problem
Packages with given weights must ship in order. Each day the ship loads packages (in order) up to its capacity. Find the **minimum capacity** to ship everything within `days`.

```
Input:  weights = [1,2,3,4,5,6,7,8,9,10], days = 5    Output: 15
```

---

## Intuition: binary search on capacity
- Minimum possible capacity = `max(weights)` (must carry the heaviest single package).
- Maximum = `sum(weights)` (everything in one day).
- A larger capacity needs fewer days (monotonic). Binary search for the smallest capacity that ships within `days`.

Feasibility: greedily load packages day by day, starting a new day when the next package would exceed capacity.

---

## Code (C++)
```cpp
int daysNeeded(vector<int>& w, int cap) {
    int days = 1, load = 0;
    for (int x : w) {
        if (load + x > cap) { days++; load = 0; }   // start a new day
        load += x;
    }
    return days;
}

int shipWithinDays(vector<int>& weights, int days) {
    int low = *max_element(weights.begin(), weights.end());
    int high = accumulate(weights.begin(), weights.end(), 0);
    int ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (daysNeeded(weights, mid) <= days) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log(sum − max)) |
| Space | O(1) |

> The lower bound **must** be `max(weights)` — a capacity below that can never carry the heaviest package.
