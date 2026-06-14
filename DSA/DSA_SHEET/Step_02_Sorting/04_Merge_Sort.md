# Merge Sort  🟡

**Reference:** [GeeksforGeeks — Merge Sort](https://www.geeksforgeeks.org/merge-sort/) · [LeetCode 912 — Sort an Array](https://leetcode.com/problems/sort-an-array/)

---

## Problem
Sort an array of N integers using the **Merge Sort** (divide and conquer) algorithm.

**Example**
```
Input:  arr = [3, 1, 2, 4, 1, 5, 2, 6, 4]
Output: [1, 1, 2, 2, 3, 4, 4, 5, 6]
```

---

## Intuition
Divide and conquer:
1. **Divide** the array into two halves around the middle.
2. **Recursively sort** each half.
3. **Merge** the two sorted halves into one sorted array using a two-pointer merge.

The base case is a subarray of size 1 (already sorted).

---

## Code (C++)
```cpp
void merge(vector<int>& arr, int low, int mid, int high) {
    vector<int> temp;
    int left = low, right = mid + 1;

    // merge the two sorted halves [low..mid] and [mid+1..high]
    while (left <= mid && right <= high) {
        if (arr[left] <= arr[right]) temp.push_back(arr[left++]);
        else                         temp.push_back(arr[right++]);
    }
    while (left <= mid)  temp.push_back(arr[left++]);   // leftovers
    while (right <= high) temp.push_back(arr[right++]);

    for (int i = low; i <= high; i++)                  // copy back
        arr[i] = temp[i - low];
}

void mergeSort(vector<int>& arr, int low, int high) {
    if (low >= high) return;                           // base case: size 1
    int mid = low + (high - low) / 2;
    mergeSort(arr, low, mid);                          // sort left
    mergeSort(arr, mid + 1, high);                     // sort right
    merge(arr, low, mid, high);                        // merge halves
}

// call: mergeSort(arr, 0, arr.size() - 1);
```

---

## How it works
```
[3,1,2,4,1]  -->  split  -->  [3,1]   [2,4,1]
                              /  \      /   \
                            [3] [1]  [2]  [4,1]
                            merge     merge  ...
                            [1,3]    [2] [1,4]
                                   merge -> [1,2,4]
              merge [1,3] + [1,2,4] -> [1,1,2,3,4]
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) — log N levels × O(N) merge per level (best = worst = avg) |
| Space | O(N) — the `temp` array; plus O(log N) recursion stack |
| Stable? | ✅ Yes (use `<=` in the merge to keep equal elements' order) |

> Merge sort is the basis for **counting inversions** and **count reverse pairs** (Step 3 hard problems) — you count cross-pairs during the merge step.
