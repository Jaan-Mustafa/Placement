# Remove Duplicates from Sorted Array  🟢

**Reference:** [LeetCode 26 — Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

---

## Problem
Given a **sorted** array, remove duplicates **in place** so each element appears once. Return the count of unique elements `k`; the first `k` positions must hold the unique values.

```
Input:  [1, 1, 2, 2, 2, 3]    Output: k = 3, arr = [1, 2, 3, ...]
```

---

## Intuition
**Two pointers.** `i` marks the last unique element's position; `j` scans ahead. When `arr[j]` differs from `arr[i]`, it's a new unique value — advance `i` and copy it there. Because the array is sorted, duplicates are adjacent.

---

## Code (C++)
```cpp
int removeDuplicates(vector<int>& arr) {
    if (arr.empty()) return 0;
    int i = 0;                          // last unique index
    for (int j = 1; j < arr.size(); j++) {
        if (arr[j] != arr[i]) {         // new unique value found
            i++;
            arr[i] = arr[j];            // place it next
        }
    }
    return i + 1;                       // count of uniques
}
```

---

## Dry run
`[1,1,2,2,2,3]`: i=0; j finds 2 → i=1,arr=[1,2,...]; finds 3 → i=2,arr=[1,2,3,...]. Returns **3**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) in place |

> Works only because the array is **sorted**. For unsorted input, use a hash set (O(N) time, O(N) space).
