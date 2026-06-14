# N Meetings in One Room  🟡

**Reference:** [GeeksforGeeks — N meetings in one room](https://www.geeksforgeeks.org/find-maximum-meetings-in-one-room/)

---

## Problem
Given start times `start[]` and end times `end[]` of `N` meetings, only one meeting can run at a time. Find the maximum number of meetings the room can hold.

```
Input:  start = [1,3,0,5,8,5], end = [2,4,6,7,9,9]
Output: 4   (meetings ending at 2,4,7,9)
```

---

## Intuition
This is **activity selection**. Sort meetings by **end time** ascending. Greedily pick a meeting if it starts strictly after the last selected meeting's end. Choosing the earliest-finishing compatible meeting leaves the most room for the rest — an exchange argument shows it's always safe.

---

## Code (C++)
```cpp
int maxMeetings(vector<int>& start, vector<int>& end) {
    int n = start.size();
    vector<pair<int,int>> mt(n);       // (end, start)
    for (int i = 0; i < n; i++) mt[i] = {end[i], start[i]};
    sort(mt.begin(), mt.end());        // sort by end time ascending

    int count = 0, lastEnd = -1;
    for (auto& [e, s] : mt) {
        if (s > lastEnd) {             // starts after last meeting ends
            count++;
            lastEnd = e;               // occupy room until this end
        }
    }
    return count;
}
```

---

## Dry run
Sorted by end: (2,1),(4,3),(6,0),(7,5),(9,8),(9,5). Pick 2; 3>2 pick 4; 0<4 skip; 5>4 pick 7; 8>7 pick 9; 5<9 skip. Count **4**.

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(N) |

> ⚠️ Use `s > lastEnd` (strict) if a meeting ending at time `t` blocks one starting at `t`. If meetings can share an endpoint, use `s >= lastEnd`. Check the problem's convention.
