# Asteroid Collision  🟡

**Reference:** [LeetCode 735 — Asteroid Collision](https://leetcode.com/problems/asteroid-collision/)

---

## Problem
Each asteroid has a size (magnitude) and direction (sign: positive = right, negative = left). When two moving toward each other collide, the smaller explodes; equal sizes both explode. Return the surviving asteroids.

```
Input:  [5, 10, -5]   Output: [5, 10]   (-5 destroyed by 10)
Input:  [8, -8]       Output: []         (both equal -> destroyed)
Input:  [10, 2, -5]   Output: [10]       (-5 kills 2, then dies to 10)
```

---

## Intuition
Collisions only happen when a right-moving asteroid (positive, on the stack) meets a left-moving one (negative, incoming). Use a **stack** of survivors. For each negative asteroid, resolve it against positive tops: smaller positives get popped, equal magnitudes cancel both, and a bigger positive destroys the incoming one. If nothing positive blocks it, the negative survives onto the stack.

---

## Code (C++)
```cpp
vector<int> asteroidCollision(vector<int>& asteroids) {
    vector<int> st;                          // use vector as a stack
    for (int a : asteroids) {
        bool alive = true;
        // collision only if incoming is negative and top is positive (moving right)
        while (alive && a < 0 && !st.empty() && st.back() > 0) {
            if (st.back() < -a) {            // top smaller -> it explodes
                st.pop_back();
                continue;
            } else if (st.back() == -a) {    // equal -> both explode
                st.pop_back();
            }
            alive = false;                   // incoming destroyed (or both)
        }
        if (alive) st.push_back(a);
    }
    return st;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each asteroid pushed/popped at most once |
| Space | O(N) |

> ⚠️ Two same-direction asteroids never collide; only a positive-on-stack vs negative-incoming pair does. Handle the equal-magnitude case by popping the top *and* killing the incoming.
