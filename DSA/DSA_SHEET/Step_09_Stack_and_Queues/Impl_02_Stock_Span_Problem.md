# Stock Span Problem  🟡

**Reference:** [LeetCode 901 — Online Stock Span](https://leetcode.com/problems/online-stock-span/) · [GeeksforGeeks](https://www.geeksforgeeks.org/the-stock-span-problem/)

---

## Problem
The span of a stock's price on a day is the number of consecutive days (up to and including today) the price was ≤ today's price. Compute the span for each day.

```
Input:  [100, 80, 60, 70, 60, 75, 85]
Output: [1, 1, 1, 2, 1, 4, 6]
```

---

## Intuition
Today's span jumps over all previous days whose price is ≤ today's. A **monotonic decreasing stack of (price, span)** lets us skip them in bulk: pop every entry with price ≤ today's, accumulating their spans, then push today with the summed span. This makes each day amortised O(1).

---

## Code (C++)
```cpp
class StockSpanner {
    stack<pair<int,int>> st;                // (price, span), decreasing prices
public:
    int next(int price) {
        int span = 1;
        // absorb spans of all not-greater previous prices
        while (!st.empty() && st.top().first <= price) {
            span += st.top().second;
            st.pop();
        }
        st.push({price, span});
        return span;
    }
};

// Batch version:
vector<int> calculateSpan(vector<int>& prices) {
    StockSpanner s;
    vector<int> res;
    for (int p : prices) res.push_back(s.next(p));
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) amortised overall |
| Space | O(N) |

> This is "previous greater element" in disguise: the span counts days back to the nearest strictly greater price. Storing accumulated spans on the stack avoids re-scanning popped days.
