# Count Subarrays with Sum K  🟡

**Reference:** [LeetCode 560 — Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

---

## Problem
Count the number of contiguous subarrays whose sum equals K. The array may contain negatives.

```
Input:  [1, 1, 1], K = 2    Output: 2
Input:  [1, 2, 3], K = 3    Output: 2   ([1,2] and [3])
```

---

## Intuition: prefix sum + hash map
A subarray `(j+1..i)` sums to K iff `prefix[i] - prefix[j] = K`, i.e. `prefix[j] = prefix[i] - K`. So while sweeping and maintaining a running `sum`, the number of subarrays ending at `i` with sum K equals how many times `sum - K` has occurred as a prefix before.

Store a map of `prefixSum → count`. Seed it with `{0: 1}` to count subarrays starting at index 0.

---

## Code (C++)
```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int,int> cnt;
    cnt[0] = 1;                            // empty prefix
    int sum = 0, ans = 0;
    for (int x : nums) {
        sum += x;
        if (cnt.count(sum - k))
            ans += cnt[sum - k];           // add subarrays ending here
        cnt[sum]++;                        // record this prefix sum
    }
    return ans;
}
```

---

## Why seed `{0:1}`?
If the prefix sum itself equals K, the subarray from index 0 is valid — that corresponds to `sum - K = 0`, which must already be in the map.

## Complexity
| | |
|---|---|
| Time | O(N) average |
| Space | O(N) |

> ⚠️ This **counts** subarrays. For the **longest** subarray with sum K (negatives allowed), store the earliest index instead of a count — see Step 3 Easy_14.
