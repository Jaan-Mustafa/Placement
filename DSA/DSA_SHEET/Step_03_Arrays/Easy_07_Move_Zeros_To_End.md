# Move Zeros to End  🟢

**Reference:** [LeetCode 283 — Move Zeroes](https://leetcode.com/problems/move-zeroes/)

---

## Problem
Move all 0s to the end of the array while keeping the relative order of the non-zero elements. Do it in place.

```
Input:  [0, 1, 0, 3, 12]    Output: [1, 3, 12, 0, 0]
```

---

## Intuition
**Two pointers.** `j` points to the position where the next non-zero should go. Scan with `i`; whenever `arr[i]` is non-zero, swap it into position `j` and advance `j`. All zeros naturally bubble to the back, order preserved.

---

## Code (C++)
```cpp
void moveZeroes(vector<int>& arr) {
    int j = 0;                          // next slot for a non-zero
    for (int i = 0; i < arr.size(); i++) {
        if (arr[i] != 0) {
            swap(arr[i], arr[j]);
            j++;
        }
    }
}
```

---

## Dry run
`[0,1,0,3,12]`: i=1 (1) swap→[1,0,0,3,12],j=1; i=3 (3) swap→[1,3,0,0,12],j=2; i=4 (12) swap→[1,3,12,0,0],j=3. ✅

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
