# Minimum Platforms  🟡

**Reference:** [GeeksforGeeks — Minimum number of platforms](https://www.geeksforgeeks.org/minimum-number-platforms-required-railwaybus-station/)

---

## Problem
Given arrival times `arr[]` and departure times `dep[]` of trains at a station, find the minimum number of platforms required so that no train waits.

```
Input:  arr = [900, 940, 950, 1100, 1500, 1800]
        dep = [910, 1200, 1120, 1130, 1900, 2000]
Output: 3
```

---

## Intuition
Sort arrivals and departures **independently**. Use two pointers and sweep through events in time order: every arrival before the next departure needs a new platform (`platforms++`); when a departure happens first, a platform frees up (`platforms--`). The running maximum of concurrently occupied platforms is the answer.

---

## Code (C++)
```cpp
int findPlatform(vector<int>& arr, vector<int>& dep) {
    int n = arr.size();
    sort(arr.begin(), arr.end());
    sort(dep.begin(), dep.end());

    int platforms = 0, maxPlat = 0;
    int i = 0, j = 0;                  // i -> arrivals, j -> departures
    while (i < n) {
        if (arr[i] <= dep[j]) {        // a train arrives before/with next departure
            platforms++;               // needs a platform
            i++;
            maxPlat = max(maxPlat, platforms);
        } else {                       // a train departs first
            platforms--;               // frees a platform
            j++;
        }
    }
    return maxPlat;
}
```

---

## Dry run
Peak overlap occurs around 950–1100 when trains arriving at 900/940/950 haven't all departed → 3 platforms needed. Answer **3**.

## Complexity
| | |
|---|---|
| Time | O(N log N) (sorting) |
| Space | O(1) extra |

> ⚠️ Use `arr[i] <= dep[j]` (inclusive): if a train arrives exactly when another departs, they still need separate platforms in the standard convention.
