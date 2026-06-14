# Shortest Job First (SJF)  🟡

**Reference:** [GeeksforGeeks — Shortest Job First (non-preemptive) average waiting time](https://www.geeksforgeeks.org/program-for-shortest-job-first-or-sjf-cpu-scheduling-set-1-non-preemptive/)

---

## Problem
Given the burst (execution) times of processes that are all available at time 0, compute the **average waiting time** under non-preemptive Shortest Job First scheduling.

```
Input:  burst = [4, 3, 7, 1, 2]
Output: 4   (average waiting time)
```

---

## Intuition
Waiting time of a job = sum of burst times of all jobs scheduled before it. To minimize total (and hence average) waiting time, **run the shortest jobs first** — short jobs finishing early reduce the cumulative wait imposed on everyone after them. Sort by burst time ascending; a running total of completed time gives each job's wait.

---

## Code (C++)
```cpp
double sjfAverageWaiting(vector<int> burst) {
    sort(burst.begin(), burst.end());  // shortest job first
    long long waitSum = 0, elapsed = 0;
    for (int b : burst) {
        waitSum += elapsed;            // this job waited for all previous jobs
        elapsed += b;                  // then it runs for b units
    }
    return (double)waitSum / burst.size();
}
```

---

## Dry run
Sorted: [1,2,3,4,7]. Waits: 0,1,3,6,10 → sum 20, n=5 → average **4**.

## Complexity
| | |
|---|---|
| Time | O(N log N) (sorting) |
| Space | O(1) extra |

> ⚠️ This assumes all jobs arrive at time 0. With differing arrival times you must pick the shortest job *among those already arrived* at each step (use a min-heap), which changes the algorithm.
