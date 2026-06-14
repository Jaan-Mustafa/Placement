# Job Sequencing Problem  🟡

**Reference:** [GeeksforGeeks — Job Sequencing Problem](https://www.geeksforgeeks.org/job-sequencing-problem/)

---

## Problem
Given `N` jobs, each with a `deadline` and a `profit`. Each job takes one unit of time and earns its profit only if completed on or before its deadline. One job runs per time slot. Maximize total profit (and report the count of jobs done).

```
Input:  jobs = [(id=1,dl=4,p=20),(2,1,10),(3,1,40),(4,1,30)]
Output: 2 jobs, profit 60   (do job 3 at t=1, job 1 at t=2..4)
```

---

## Intuition
**Sort jobs by profit descending.** For each job, schedule it in the **latest free slot at or before its deadline** — leaving earlier slots open for other tight-deadline jobs. Taking the most profitable jobs first, and placing them as late as legally possible, maximizes how many high-value jobs fit.

---

## Code (C++)
```cpp
struct Job { int id, deadline, profit; };

pair<int,int> jobSequencing(vector<Job>& jobs) {
    sort(jobs.begin(), jobs.end(), [](const Job& a, const Job& b) {
        return a.profit > b.profit;        // highest profit first
    });

    int maxDeadline = 0;
    for (auto& j : jobs) maxDeadline = max(maxDeadline, j.deadline);

    vector<int> slot(maxDeadline + 1, -1); // slot[t] = job id placed at time t (1-indexed)
    int countDone = 0, totalProfit = 0;

    for (auto& j : jobs) {
        // find latest free slot at or before this job's deadline
        for (int t = j.deadline; t >= 1; t--) {
            if (slot[t] == -1) {
                slot[t] = j.id;
                countDone++;
                totalProfit += j.profit;
                break;
            }
        }
    }
    return {countDone, totalProfit};
}
```

---

## Dry run
Sorted by profit: (3,1,40),(4,1,30),(1,4,20),(2,1,10). Job3 → slot1; Job4 dl1 occupied → skip; Job1 dl4 → slot4; Job2 dl1 occupied → skip. **2 jobs, profit 60**.

## Complexity
| | |
|---|---|
| Time | O(N log N + N·D) (D = max deadline) |
| Space | O(D) for the slot array |

> ⚠️ Filling the **latest** available slot (loop deadline → 1) is the key. Greedily grabbing the earliest slot can block later jobs with tight deadlines. A DSU on slots reduces the inner scan to near O(1).
