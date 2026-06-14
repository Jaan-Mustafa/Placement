# Connect N Ropes with Minimum Cost  🔴

**Reference:** [GeeksforGeeks — Connect n ropes with minimum cost](https://www.geeksforgeeks.org/connect-n-ropes-minimum-cost/)

---

## Problem
You have ropes of given lengths. Connecting two ropes costs the **sum of their lengths**. Connect all ropes into one and minimize total cost.

```
Input:  [4, 3, 2, 6]   Output: 29
        2+3=5 (cost 5), 4+5=9 (cost 9), 6+9=15 (cost 15) -> 5+9+15 = 29
```

---

## Intuition
This is Huffman-coding's merge: repeatedly combine the **two shortest** ropes so that small lengths are added many times and large lengths few times. A **min-heap** always hands back the two smallest. Pop two, push their sum, and accumulate that sum into the cost — until one rope remains.

---

## Code (C++)
```cpp
long long minCostToConnect(vector<int>& ropes) {
    priority_queue<long long, vector<long long>, greater<long long>> mn(
        ropes.begin(), ropes.end());            // min-heap, O(n) build

    long long cost = 0;
    while (mn.size() > 1) {
        long long a = mn.top(); mn.pop();        // two shortest
        long long b = mn.top(); mn.pop();
        cost += a + b;                           // pay to merge
        mn.push(a + b);                          // merged rope re-enters
    }
    return cost;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(N) |

> ⚠️ Use `long long` for cost and heap elements — repeated sums overflow `int` for large inputs. Greedily picking the two **smallest** (not arbitrary pairs) is what minimizes total cost; this is the Huffman argument.
