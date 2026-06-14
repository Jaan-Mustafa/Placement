# Minimum Days to Make M Bouquets  🟡

**Reference:** [LeetCode 1482 — Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/)

---

## Problem
`bloomDay[i]` is the day flower `i` blooms. To make a bouquet you need `k` **adjacent** bloomed flowers. Find the minimum number of days to make `m` bouquets, or -1 if impossible.

```
Input:  bloomDay = [1,10,3,10,2], m = 3, k = 1    Output: 3
Input:  bloomDay = [1,10,3,10,2], m = 3, k = 2    Output: -1
```

---

## Intuition: binary search on the day
If `m*k > n`, it's impossible. Otherwise the answer day lies in `[min(bloomDay), max(bloomDay)]`. More days → more flowers bloomed → more bouquets possible (monotonic). Binary search for the smallest day where we can form `m` bouquets.

The feasibility check: scan flowers, count consecutive bloomed (≤ given day), forming a bouquet every `k` adjacent ones.

---

## Code (C++)
```cpp
bool canMake(vector<int>& bloom, int day, int m, int k) {
    int bouquets = 0, flowers = 0;
    for (int b : bloom) {
        if (b <= day) {                 // bloomed
            if (++flowers == k) { bouquets++; flowers = 0; }
        } else {
            flowers = 0;                // streak broken (must be adjacent)
        }
    }
    return bouquets >= m;
}

int minDays(vector<int>& bloom, int m, int k) {
    long long need = 1LL * m * k;
    if (need > bloom.size()) return -1;             // not enough flowers
    int low = *min_element(bloom.begin(), bloom.end());
    int high = *max_element(bloom.begin(), bloom.end());
    int ans = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (canMake(bloom, mid, m, k)) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log(maxDay)) |
| Space | O(1) |

> ⚠️ Reset the flower streak to 0 on an unbloomed flower — bouquets need **adjacent** flowers.
