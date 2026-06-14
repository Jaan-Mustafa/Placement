# Second Largest Element (Without Sorting)  🟢

**Reference:** [GeeksforGeeks — Second largest element](https://www.geeksforgeeks.org/find-second-largest-element-array/)

---

## Problem
Find the second largest element in an array **in one pass, without sorting**. If it doesn't exist, return -1.

```
Input:  [1, 2, 4, 7, 7, 5]    Output: 5
Input:  [1, 1, 1]             Output: -1
```

---

## Intuition
Track two variables: `largest` and `secondLargest`. For each element:
- If it's bigger than `largest` → the old largest becomes secondLargest, update largest.
- Else if it's smaller than largest but bigger than secondLargest → update secondLargest.

The `!=` check skips duplicates of the largest.

---

## Code (C++)
```cpp
int secondLargest(vector<int>& arr) {
    int largest = INT_MIN, second = INT_MIN;
    for (int x : arr) {
        if (x > largest) {
            second = largest;       // old largest demoted
            largest = x;
        } else if (x < largest && x > second) {
            second = x;
        }
    }
    return second == INT_MIN ? -1 : second;
}
```

---

## Dry run
`[1, 2, 4, 7, 7, 5]`: largest→1,2,4,7 (second→4); next 7 (==largest, skip); 5>4 → second=5. Result **5**.

## Complexity
| | |
|---|---|
| Time | O(N) single pass |
| Space | O(1) |

> Sorting would be O(N log N) — this beats it. Sorting also struggles with duplicates of the max.
