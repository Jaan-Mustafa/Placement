# Fruit Into Baskets  🟡

**Reference:** [LeetCode 904](https://leetcode.com/problems/fruit-into-baskets/)

---

## Problem
You have rows of fruit trees (`fruits[i]` = fruit type). You have **two baskets**, each holding one type only. Starting anywhere, pick one fruit per tree moving right, stopping when you can't. Return the **maximum fruits** you can collect.

This is exactly: **longest subarray with at most 2 distinct values**.

```
Input:  fruits = [1,2,1]       Output: 3   (all)
Input:  fruits = [0,1,2,2]     Output: 3   ([1,2,2])
Input:  fruits = [1,2,3,2,2]   Output: 4   ([2,3,2,2])
```

---

## Intuition
**Sliding window with at most 2 distinct keys.**
- Use a hashmap of `type -> count` for the current window.
- Expand `right`; while the map has more than 2 types, shrink from `left`, erasing a type when its count hits 0.
- Track max window size.

---

## Code (C++)
```cpp
int totalFruit(vector<int>& fruits) {
    unordered_map<int,int> cnt;          // fruit type -> count in window
    int left = 0, best = 0;
    for (int right = 0; right < (int)fruits.size(); right++) {
        cnt[fruits[right]]++;            // expand
        while (cnt.size() > 2) {         // more than 2 baskets needed -> shrink
            if (--cnt[fruits[left]] == 0)
                cnt.erase(fruits[left]);
            left++;
        }
        best = max(best, right - left + 1);
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) — map holds at most 3 keys |

> ⚠️ Always `erase` the key when its count reaches 0, otherwise `cnt.size()` overstates the distinct count.
