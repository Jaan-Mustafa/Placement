# Check String Palindrome (Recursion)  🟢

**Reference:** [GeeksforGeeks — Recursive palindrome check](https://www.geeksforgeeks.org/c-program-check-given-string-palindrome/) · [LeetCode 125 — Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

---

## Problem
Check whether a string is a palindrome (reads the same both ways) using recursion.

```
Input:  "madam"    Output: true
Input:  "abc"      Output: false
```

---

## Intuition
Compare the outermost characters (`i` and `n-1-i`). If they match, recurse inward (`i+1`). If any mismatch → not a palindrome. Base case: pointers meet in the middle → palindrome.

---

## Code (C++)
```cpp
bool isPalindrome(string& s, int i) {
    int n = s.size();
    if (i >= n / 2) return true;            // reached middle → palindrome
    if (s[i] != s[n - i - 1]) return false; // mismatch → not palindrome
    return isPalindrome(s, i + 1);          // recurse inward
}

// call: isPalindrome(s, 0);
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |

> LeetCode 125 adds: ignore non-alphanumerics and case. Pre-clean the string or skip such characters with two moving pointers.
