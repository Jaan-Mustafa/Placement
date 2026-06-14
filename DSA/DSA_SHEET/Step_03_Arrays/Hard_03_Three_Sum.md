# 3 Sum  🔴

**Reference:** [LeetCode 15 — 3Sum](https://leetcode.com/problems/3sum/)

---

## Problem
Find all **unique** triplets `[a, b, c]` that sum to 0.

```
Input:  [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
```

---

## Intuition
Sort the array. Fix the first element `i`, then use **two pointers** (`j`, `k`) on the remaining sorted range to find pairs summing to `-nums[i]`. Sorting makes duplicate-skipping easy: skip repeated values for `i`, `j`, and `k` so triplets stay unique.

---

## Code (C++)
```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    int n = nums.size();
    vector<vector<int>> res;
    for (int i = 0; i < n - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;     // skip dup i
        int j = i + 1, k = n - 1;
        while (j < k) {
            int sum = nums[i] + nums[j] + nums[k];
            if (sum < 0) j++;
            else if (sum > 0) k--;
            else {
                res.push_back({nums[i], nums[j], nums[k]});
                j++; k--;
                while (j < k && nums[j] == nums[j - 1]) j++;  // skip dup j
                while (j < k && nums[k] == nums[k + 1]) k--;  // skip dup k
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
| Time | O(N²) — outer loop × two-pointer scan |
| Space | O(1) extra (output excluded), O(log N) for sort |

> The two-pointer move works because the array is sorted: too-small sum → move left pointer right; too-large → move right pointer left.
