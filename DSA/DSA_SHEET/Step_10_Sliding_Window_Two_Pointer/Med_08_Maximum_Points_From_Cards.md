# Maximum Points You Can Obtain From Cards  🟡

**Reference:** [LeetCode 1423](https://leetcode.com/problems/maximum-points-you-can-obtain-from-cards/)

---

## Problem
You have `cardPoints` in a row. In `k` moves you take one card from **either end** each time. Maximize the total points of the `k` taken cards.

```
Input:  cardPoints = [1,2,3,4,5,6,1], k = 3   Output: 12   (take 6,1 from right + 5)
Input:  cardPoints = [2,2,2], k = 2           Output: 4
Input:  cardPoints = [9,7,7,9,7,7,9], k = 7   Output: 55   (take all)
```

---

## Intuition: minimize the leftover window
Taking `k` cards from the ends leaves a **contiguous middle window of size `n − k`**. Maximizing what we take = minimizing the sum of that leftover window.

So slide a fixed-size window of length `n − k` across the array, find its minimum sum, and subtract from the total.

(Equivalent prefix/suffix formulation: start by taking the first `k` from the left, then swap left cards for right cards one at a time, tracking the max.)

---

## Code (C++)
```cpp
int maxScore(vector<int>& cardPoints, int k) {
    int n = cardPoints.size();
    int total = 0;
    for (int x : cardPoints) total += x;

    int win = n - k;                     // leftover window size
    if (win == 0) return total;          // take everything

    int sum = 0;
    for (int i = 0; i < win; i++) sum += cardPoints[i];
    int minWindow = sum;
    // slide the fixed-size window
    for (int i = win; i < n; i++) {
        sum += cardPoints[i] - cardPoints[i - win];
        minWindow = min(minWindow, sum);
    }
    return total - minWindow;            // points taken = total - smallest leftover
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> ⚠️ Handle `k == n` (window size 0) explicitly — you simply take every card and return `total`.
