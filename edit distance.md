# Edit Distance — Notes

> **Problem:** Given two strings, find the minimum number of operations (insert, delete, replace) to convert word1 into word2. Each operation costs 1.
> LC 72 — leetcode.com/problems/edit-distance

---

## Why this problem is hard

The non-obvious insight: **you don't need to actually modify the string.**
Just move pointers. The string never changes — only `i` and `j` move.

This is the one thing nobody figures out on their own the first time.

---

## The key insight — operations as pointer movements

```
INSERT  word2[j] into word1 at position i
  → word2[j] now matched → advance j only
  → solve(i, j+1)

DELETE  word1[i]
  → word1[i] gone → advance i only
  → solve(i+1, j)

REPLACE word1[i] with word2[j]
  → both matched → advance both
  → solve(i+1, j+1)

MATCH   word1[i] == word2[j]
  → no cost → advance both
  → solve(i+1, j+1)
```

The string stays unchanged. Only pointers move.

---

## Base cases

```
i == word1.length():
  word1 exhausted — only option is INSERT remaining word2 characters
  cost = word2.length() - j

j == word2.length():
  word2 exhausted — only option is DELETE remaining word1 characters
  cost = word1.length() - i
```

---

## Version 1 — Plain Recursion

```java
private int solve(String word1, String word2, int i, int j) {
    if (i == word1.length()) return word2.length() - j;  // insert rest of word2
    if (j == word2.length()) return word1.length() - i;  // delete rest of word1

    if (word1.charAt(i) == word2.charAt(j)) {
        return solve(word1, word2, i+1, j+1);             // match — no cost
    }

    int insert  = solve(word1, word2, i,   j+1);          // insert
    int delete  = solve(word1, word2, i+1, j);            // delete
    int replace = solve(word1, word2, i+1, j+1);          // replace

    return 1 + Math.min(Math.min(insert, delete), replace);
}
```

---

## Version 2 — Memoization (Top-Down DP)

**Two things change:** `i` and `j` → `memo[i][j]`

```java
class Solution {
    int[][] memo;

    public int minDistance(String word1, String word2) {
        memo = new int[word1.length()+1][word2.length()+1];
        for (int[] arr : memo) Arrays.fill(arr, -1);
        return solve(word1, word2, 0, 0);
    }

    private int solve(String word1, String word2, int i, int j) {
        if (i == word1.length()) return word2.length() - j;
        if (j == word2.length()) return word1.length() - i;
        if (memo[i][j] != -1) return memo[i][j];          // cache hit

        if (word1.charAt(i) == word2.charAt(j)) {
            memo[i][j] = solve(word1, word2, i+1, j+1);
        } else {
            int insert  = solve(word1, word2, i,   j+1);
            int delete  = solve(word1, word2, i+1, j);
            int replace = solve(word1, word2, i+1, j+1);
            memo[i][j] = 1 + Math.min(Math.min(insert, delete), replace);
        }

        return memo[i][j];
    }
}
```

---

## Version 3 — Tabulation (Bottom-Up DP)

**Fill direction:** calls `i+1` and `j+1` → needs RIGHT and DOWN → fill RIGHT TO LEFT, BOTTOM TO TOP.

**Base cases:** fill last row and last column explicitly before the main loop.

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[][] dp = new int[m+1][n+1];

        // base cases
        for (int i = 0; i <= m; i++) dp[i][n] = m - i;  // last column
        for (int j = 0; j <= n; j++) dp[m][j] = n - j;  // last row

        // fill right to left, bottom to top
        for (int i = m-1; i >= 0; i--) {
            for (int j = n-1; j >= 0; j--) {
                if (word1.charAt(i) == word2.charAt(j)) {
                    dp[i][j] = dp[i+1][j+1];             // match — no cost
                } else {
                    dp[i][j] = 1 + Math.min(
                        Math.min(dp[i][j+1],              // insert
                                 dp[i+1][j]),             // delete
                                 dp[i+1][j+1]);           // replace
                }
            }
        }

        return dp[0][0];
    }
}
```

---

## Slot Trace — word1="horse", word2="ros"

```
       r    o    s    ""
h    [ 3,   3,   3,   5 ]   i=0
o    [ 3,   2,   3,   4 ]   i=1
r    [ 2,   2,   3,   3 ]   i=2 (wait — 'r'=='r' match → dp[2][0]=dp[3][1])
s    [ 3,   2,   1,   2 ]   i=3
e    [ 4,   3,   2,   1 ]   i=4
""   [ 3,   2,   1,   0 ]   i=5 (base case row: dp[5][j]=3-j)
      j=0  j=1  j=2  j=3   (base case col: dp[i][3]=5-i)

Fill bottom-right to top-left:

dp[4][2]: 'e'!='s' → 1+min(dp[4][3],dp[5][2],dp[5][3])=1+min(1,1,0)=1
dp[4][1]: 'e'!='o' → 1+min(dp[4][2],dp[5][1],dp[5][2])=1+min(1,2,1)=2
dp[4][0]: 'e'!='r' → 1+min(dp[4][1],dp[5][0],dp[5][1])=1+min(2,3,2)=3
dp[3][2]: 's'=='s' → dp[4][3]=1
dp[3][1]: 's'!='o' → 1+min(dp[3][2],dp[4][1],dp[4][2])=1+min(1,2,1)=2
dp[3][0]: 's'!='r' → 1+min(dp[3][1],dp[4][0],dp[4][1])=1+min(2,3,2)=3
dp[2][0]: 'r'=='r' → dp[3][1]=2
dp[1][1]: 'o'=='o' → dp[2][2]
dp[0][0]: 'h'!='r' → 1+min(dp[0][1],dp[1][0],dp[1][1])=3

dp[0][0] = 3 ✅
```

---

## 5 Questions Applied

```
1. What changes?
   → i (position in word1) AND j (position in word2)
   → memo[i][j], size [m+1][n+1]

2. Answer == return value?
   → YES — min operations IS what we return
   → track inside, return directly

3. How many choices?
   → fixed: match (1 call) OR 3 operations (3 calls)
   → no loop needed

4. Base case meaning?
   → i==m → word1 exhausted → insert n-j more chars → return n-j
   → j==n → word2 exhausted → delete m-i more chars → return m-i

5. Fill direction?
   → calls i+1, j+1 → needs DOWN and RIGHT
   → fill BOTTOM TO TOP, RIGHT TO LEFT
   → base cases: last row (dp[m][j]=n-j) and last col (dp[i][n]=m-i)
   → answer: dp[0][0]
```

---

## My Mistakes

### Mistake 1 — Actually modifying the string

```java
// ❌ actually performing insert/delete/replace on the string
int insert = solve(insert(word1, word2.charAt(j), i), word2, i+1, j+1);
int delete = solve(delete(word1, i), word2, i, j);
int replace = solve(replace(word1, word2.charAt(j), i), word2, i+1, j+1);
```

**Why wrong:** creates exponential string copies, indices shift after modification, loses the DP property — `solve(i,j)` must always mean the same thing regardless of how you got there.

**The insight:** you don't need to modify the string. Operations are just pointer movements:
```java
// ✅ no string modification — just move pointers
int insert  = solve(word1, word2, i,   j+1);
int delete  = solve(word1, word2, i+1, j);
int replace = solve(word1, word2, i+1, j+1);
```

**Why this works:**
```
INSERT word2[j] before word1[i] → word2[j] matched → j advances, i stays
DELETE word1[i]                 → word1[i] gone    → i advances, j stays
REPLACE word1[i] with word2[j]  → both matched     → both advance
```

### Mistake 2 — Wrong sentinel for memo

```java
if (memo[i][j] != 0) return memo[i][j];  // ❌ 0 is a valid answer (identical strings)
if (memo[i][j] != -1) return memo[i][j]; // ✅ -1 never a valid answer
```

When `word1 == word2`, edit distance is 0. Using 0 as sentinel means matching positions get recomputed every time.

### Mistake 3 — Memo array too small

```java
memo = new int[word1.length()][word2.length()];    // ❌ crashes on empty strings
memo = new int[word1.length()+1][word2.length()+1]; // ✅ safe
```

---

## Edit Distance vs LCS — comparison

Both use two string pointers `i` and `j`. Both are 2D DP. But:

| | LCS | Edit Distance |
|---|---|---|
| Match case | `1 + solve(i+1, j+1)` | `solve(i+1, j+1)` (no cost) |
| Mismatch | `max(skip i, skip j)` | `1 + min(insert, delete, replace)` |
| No match choices | 2 (skip either) | 3 (insert, delete, replace) |
| Base case | `0` (no more chars) | `remaining chars` (cost to finish) |
| Goal | maximize length | minimize operations |

LCS skips characters to find the longest common part.
Edit Distance transforms one string into another with minimum cost.

---

## One Line to Remember

> Operations are pointer movements not string modifications. Match → advance both free. No match → try insert (j+1), delete (i+1), replace (i+1,j+1), take minimum + 1.
