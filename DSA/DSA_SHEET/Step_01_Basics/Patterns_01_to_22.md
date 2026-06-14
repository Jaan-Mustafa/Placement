# Pattern Problems (1–22)  🟢

**Reference:** [takeUforward — Pattern problems](https://takeuforward.org/strivers-a2z-dsa-course/must-do-pattern-problems-before-starting-dsa/)

Printing patterns builds **loop intuition**: the outer loop controls rows, the inner loop(s) control what's printed per row. The key skill is relating the row index `i` to the count of symbols/spaces.

---

## The mental model
For every pattern ask three questions:
1. How many **rows**? → outer loop count.
2. For row `i`, how many **columns / symbols**? → inner loop bound (often a function of `i`).
3. What is **printed** at `(i, j)` — star, number, space, or alphabet?

---

## A few representative patterns

### Pattern: solid square of stars (n×n)
```cpp
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) cout << "* ";
    cout << "\n";
}
```

### Pattern: right-angled triangle
```
*
* *
* * *
```
```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) cout << "* ";   // i stars in row i
    cout << "\n";
}
```

### Pattern: number triangle
```
1
1 2
1 2 3
```
```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) cout << j << " ";
    cout << "\n";
}
```

### Pattern: pyramid (centered stars)
```
   *
  ***
 *****
```
```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 0; j < n - i; j++) cout << " ";        // leading spaces
    for (int j = 0; j < 2 * i - 1; j++) cout << "*";     // 2i-1 stars
    cout << "\n";
}
```

### Pattern: inverted triangle
```cpp
for (int i = n; i >= 1; i--) {
    for (int j = 1; j <= i; j++) cout << "* ";
    cout << "\n";
}
```

---

## All 22 patterns
The full set (squares, triangles, pyramids, diamonds, number/alphabet variants, hollow patterns, butterfly, etc.) follows the same row→inner-loop logic. Work through each on the takeUforward link above — the goal is fluency with nested loops before moving to arrays.

## Complexity
Each pattern is O(rows × cols) time, O(1) extra space.
