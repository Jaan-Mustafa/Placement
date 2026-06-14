# Longest Subarray with Sum K (Positives + Negatives)  🟡

**Reference:** [GeeksforGeeks — Longest subarray with sum K (handles negatives)](https://www.geeksforgeeks.org/longest-sub-array-sum-k/) · [LeetCode 325 (premium)](https://leetcode.com/problems/maximum-size-subarray-sum-equals-k/)

---

## Problem
Given an array that may contain **negative** numbers and a target K, find the length of the longest subarray summing to K.

```
Input:  [-1, 1, 1, -1, 2, 3], K = 3    Output: 4
Input:  [10, -5, 5, 3], K = 3          Output: 3   ([-5,5,3])
```

---

## Intuition: prefix sum + hash map
Let `prefix[i]` = sum of elements up to `i`. A subarray `(j+1..i)` sums to K iff `prefix[i] - prefix[j] = K`, i.e. `prefix[j] = prefix[i] - K`.

So as we sweep, we store the **earliest index** where each prefix sum first occurred. At index `i`, if `prefix - K` was seen before, the subarray from just after that index to `i` sums to K — take the longest such.

Sliding window fails here because adding a negative number can decrease the sum, breaking the monotonic shrink/grow assumption.

---

## Code (C++)
```cpp
int longestSubarrayWithSumK(vector<int>& a, long long k) {
    unordered_map<long long,int> firstSeen;   // prefixSum -> earliest index
    long long sum = 0;
    int best = 0;
    for (int i = 0; i < a.size(); i++) {
        sum += a[i];

        if (sum == k)                          // whole prefix works
            best = max(best, i + 1);

        if (firstSeen.count(sum - k))          // a subarray ending at i sums to k
            best = max(best, i - firstSeen[sum - k]);

        if (!firstSeen.count(sum))             // store EARLIEST occurrence only
            firstSeen[sum] = i;
    }
    return best;
}
```

---

## Why store only the earliest index?
We want the **longest** subarray, so for any given prefix sum we keep the leftmost index where it appeared — that maximizes `i - j`.

## Complexity
| | |
|---|---|
| Time | O(N) average (hash map) |
| Space | O(N) for the map |

> This prefix-sum-in-a-map pattern also powers "subarray sum equals K (count)" and "longest subarray with 0 sum".
