# Hand of Straights  🟡

**Reference:** [LeetCode 846 — Hand of Straights](https://leetcode.com/problems/hand-of-straights/)

---

## Problem
Can the `hand` be rearranged into groups of exactly `groupSize` **consecutive** cards? Return true/false.

```
Input:  hand = [1,2,3,6,2,3,4,7,8], groupSize = 3   Output: true
        groups: [1,2,3] [2,3,4] [6,7,8]
Input:  hand = [1,2,3,4,5], groupSize = 4           Output: false
```

---

## Intuition
If `n % groupSize != 0` it is impossible. Otherwise, always start a group from the **smallest remaining card** (a min-heap top): it must be a group's first card. From that card, require the next `groupSize-1` consecutive values to exist with sufficient count. Track remaining counts in a map; pop exhausted cards off the heap.

---

## Code (C++)
```cpp
bool isNStraightHand(vector<int>& hand, int groupSize) {
    int n = hand.size();
    if (n % groupSize != 0) return false;          // can't partition evenly

    map<int,int> cnt;                              // value -> remaining count
    for (int x : hand) cnt[x]++;

    priority_queue<int, vector<int>, greater<int>> mn(hand.begin(), hand.end());
    while (!mn.empty()) {
        int start = mn.top();
        if (cnt[start] == 0) { mn.pop(); continue; } // already used up
        // build a consecutive run of length groupSize starting at 'start'
        for (int v = start; v < start + groupSize; v++) {
            if (cnt[v] == 0) return false;          // missing consecutive card
            cnt[v]--;
        }
    }
    return true;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(N) |

> ⚠️ Always anchor a group at the current minimum — it cannot be the middle/end of any group, so it must lead one. Lazily skip heap entries whose count has dropped to 0 instead of trying to delete from the heap (heaps can't remove arbitrary elements).
