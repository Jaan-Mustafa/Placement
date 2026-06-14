# Count Subarrays with Given XOR K  🔴

**Reference:** [GeeksforGeeks — Subarrays with XOR K](https://www.geeksforgeeks.org/count-number-subarrays-given-xor/) · [InterviewBit](https://www.interviewbit.com/problems/subarray-with-given-xor/)

---

## Problem
Count contiguous subarrays whose XOR of all elements equals K.

```
Input:  [4, 2, 2, 6, 4], K = 6    Output: 4
```

---

## Intuition: prefix XOR + hash map
Let `xr` = XOR of elements up to index `i`. A subarray `(j+1..i)` has XOR K iff `prefXor[j] ^ prefXor[i] = K`, i.e. `prefXor[j] = prefXor[i] ^ K` (because XOR is its own inverse). So count how many earlier prefixes equal `xr ^ K`.

Seed the map with `{0: 1}` for subarrays starting at index 0.

---

## Code (C++)
```cpp
int subarraysWithXorK(vector<int>& a, int k) {
    unordered_map<int,int> cnt;
    cnt[0] = 1;                            // empty prefix
    int xr = 0, ans = 0;
    for (int x : a) {
        xr ^= x;                           // running prefix XOR
        int need = xr ^ k;                 // prefix we'd need earlier
        if (cnt.count(need))
            ans += cnt[need];
        cnt[xr]++;
    }
    return ans;
}
```

---

## Why `prefXor[j] = xr ^ K`
From `prefXor[j] ^ xr = K`, XOR both sides by `xr`: `prefXor[j] = K ^ xr`. The count of such previous prefixes is exactly the number of valid subarrays ending at `i`.

## Complexity
| | |
|---|---|
| Time | O(N) average |
| Space | O(N) |

> Mirror of "subarray sum equals K" — replace `+`/`-` with `^`, and the inverse of XOR is XOR itself.
