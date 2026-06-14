# 4 Sum  🔴

**Reference:** [LeetCode 18 — 4Sum](https://leetcode.com/problems/4sum/)

---

## Problem
Find all **unique** quadruplets summing to a given target.

```
Input:  nums = [1, 0, -1, 0, -2, 2], target = 0
Output: [[-2,-1,1,2], [-2,0,0,2], [-1,0,0,1]]
```

---

## Intuition
Extend 3Sum by one more loop. Sort, fix two elements `i` and `j`, then two-pointer (`k`, `l`) for the remaining pair. Skip duplicates at every level. Use `long long` for the running sum to avoid overflow (targets can push past int range).

---

## Code (C++)
```cpp
vector<vector<int>> fourSum(vector<int>& nums, int target) {
    sort(nums.begin(), nums.end());
    int n = nums.size();
    vector<vector<int>> res;
    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        for (int j = i + 1; j < n - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;
            int k = j + 1, l = n - 1;
            while (k < l) {
                long long sum = (long long)nums[i] + nums[j] + nums[k] + nums[l];
                if (sum < target) k++;
                else if (sum > target) l--;
                else {
                    res.push_back({nums[i], nums[j], nums[k], nums[l]});
                    k++; l--;
                    while (k < l && nums[k] == nums[k - 1]) k++;
                    while (k < l && nums[l] == nums[l + 1]) l--;
                }
            }
        }
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N³) |
| Space | O(1) extra (output excluded) |

> The pattern generalizes: **k-Sum** = (k-2) nested loops + a final two-pointer pass, giving O(N^(k-1)).
