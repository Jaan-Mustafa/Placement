# Largest Subarray with 0 Sum  🔴

**Reference:** [GeeksforGeeks — Largest subarray with 0 sum](https://www.geeksforgeeks.org/find-the-largest-subarray-with-0-sum/)

---

## Problem
Find the length of the longest contiguous subarray whose elements sum to 0.

```
Input:  [15, -2, 2, -8, 1, 7, 10, 23]    Output: 5   ([-2,2,-8,1,7])
```

---

## Intuition: prefix sum + hash map
If the prefix sum repeats at indices `j` and `i` (`j < i`), the subarray `(j+1..i)` sums to 0. Store the **earliest index** of each prefix sum; when the same sum reappears, the gap is a zero-sum subarray. Track the longest such gap. A prefix sum of 0 itself means the subarray from index 0 works.

---

## Code (C++)
```cpp
int maxLen(vector<int>& a) {
    unordered_map<long long,int> firstIdx;   // prefixSum -> earliest index
    long long sum = 0;
    int best = 0;
    for (int i = 0; i < a.size(); i++) {
        sum += a[i];
        if (sum == 0)
            best = max(best, i + 1);          // whole prefix sums to 0
        else if (firstIdx.count(sum))
            best = max(best, i - firstIdx[sum]);  // repeated sum -> zero gap
        else
            firstIdx[sum] = i;                // store earliest only
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) average |
| Space | O(N) |

> Same prefix-sum-map skeleton as "longest subarray sum K" — here the target is 0, equivalent to two equal prefix sums.
