# Longest Common Subsequence — Notes

> **Problem:** Given two strings, find the length of their longest common subsequence.
> LC 1143 — leetcode.com/problems/longest-common-subsequence

---

## What is a subsequence

Characters in order but not necessarily adjacent.
`"ace"` is a subsequence of `"abcde"` — pick a, skip b, pick c, skip d, pick e.

---

## Why this needs 2D DP

Every previous problem had ONE changing parameter.
This has TWO — position in text1 (`i`) and position in text2 (`j`).

```
climb(n)          → memo[n]        1D
rob(i)            → memo[i]        1D
coinChange(amt)   → memo[amt]      1D
LCS(i, j)         → memo[i][j]    2D  ← two things change
```

---

## The choice at every position

```
text1[i] == text2[j]  →  MATCH
  this character is in LCS
  return 1 + LCS(i+1, j+1)

text1[i] != text2[j]  →  NO MATCH
  try skipping text1[i] → LCS(i+1, j)
  try skipping text2[j] → LCS(i, j+1)
  return max of both
```

---

## Version 1 — Plain Recursion

```java
private int function(String text1, String text2, int i, int j) {
    if (i >= text1.length() || j >= text2.length()) return 0;  // base case

    if (text1.charAt(i) == text2.charAt(j)) {
        return 1 + function(text1, text2, i+1, j+1);  // match
    } else {
        return Math.max(
            function(text1, text2, i+1, j),   // skip text1[i]
            function(text1, text2, i, j+1)    // skip text2[j]
        );
    }
}
```

---

## Version 2 — Memoization (Top-Down DP)

```java
class Solution {
    int[][] memo;

    public int longestCommonSubsequence(String text1, String text2) {
        memo = new int[text1.length()][text2.length()];
        for (int[] arr : memo) Arrays.fill(arr, -1);
        return function(text1, text2, 0, 0);
    }

    private int function(String text1, String text2, int i, int j) {
        if (i >= text1.length() || j >= text2.length()) return 0;
        if (memo[i][j] != -1) return memo[i][j];  // cache hit

        if (text1.charAt(i) == text2.charAt(j)) {
            memo[i][j] = 1 + function(text1, text2, i+1, j+1);
        } else {
            memo[i][j] = Math.max(
                function(text1, text2, i+1, j),
                function(text1, text2, i, j+1)
            );
        }
        return memo[i][j];
    }
}
```

---

## Version 3 — Tabulation (Bottom-Up DP)

**Fill direction:** calls `i+1` and `j+1` → needs RIGHT and DOWN already filled → fill RIGHT TO LEFT, BOTTOM TO TOP.

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m+1][n+1];  // +1 for base case row and col

        for (int i = m-1; i >= 0; i--) {          // right to left
            for (int j = n-1; j >= 0; j--) {      // right to left
                if (text1.charAt(i) == text2.charAt(j)) {
                    dp[i][j] = 1 + dp[i+1][j+1];
                } else {
                    dp[i][j] = Math.max(dp[i+1][j], dp[i][j+1]);
                }
            }
        }

        return dp[0][0];
    }
}
```

**Why `[m+1][n+1]`:** when `i = m-1` and match occurs, you access `dp[m][j+1]` — needs row m to exist. Extra row and column hold base case value `0` automatically from array initialization.

---

## Slot Trace — text1="abcde", text2="ace"

```
     j=0  j=1  j=2  j=3
      a    c    e    ""
i=0 a[ 3,   2,   1,   0 ]
i=1 b[ 2,   2,   1,   0 ]
i=2 c[ 2,   2,   1,   0 ]
i=3 d[ 1,   1,   1,   0 ]
i=4 e[ 1,   1,   1,   0 ]
i=5""[ 0,   0,   0,   0 ]

Fill order: bottom-right to top-left

dp[4][2]: 'e'=='e' → 1+dp[5][3]=1+0=1
dp[4][1]: 'e'!='c' → max(dp[5][1],dp[4][2])=max(0,1)=1
dp[4][0]: 'e'!='a' → max(dp[5][0],dp[4][1])=max(0,1)=1
dp[3][2]: 'd'!='e' → max(dp[4][2],dp[3][3])=max(1,0)=1
dp[2][1]: 'c'=='c' → 1+dp[3][2]=1+1=2
dp[1][0]: 'b'!='a' → max(dp[2][0],dp[1][1])=max(2,2)=2
dp[0][0]: 'a'=='a' → 1+dp[1][1]=1+2=3

dp[0][0] = 3 ✅
```

---

## 5 Questions Applied

```
1. What changes between calls?
   → i (position in text1) AND j (position in text2)
   → memo[i][j], size [m][n]

2. Answer == return value?
   → YES — LCS length IS what we return
   → track inside, return directly

3. How many choices?
   → fixed: match (1 call) OR skip (2 calls)
   → no loop needed

4. Base case meaning?
   → i>=m OR j>=n → reached end of either string → 0

5. Fill direction?
   → calls i+1, j+1 → f(i+something), f(j+something)
   → needs RIGHT and DOWN → fill RIGHT TO LEFT, BOTTOM TO TOP
   → base cases: last row and column (auto 0 from initialization)
   → answer: dp[0][0]
```

---

## 1D vs 2D DP — The Pattern

```
ONE parameter changes   → 1D memo → one loop in tabulation
TWO parameters change  → 2D memo → two nested loops in tabulation
```

```java
// 1D — climbing stairs
int[] dp = new int[n+1];
for (int i = 2; i <= n; i++) { dp[i] = dp[i-1] + dp[i-2]; }

// 2D — LCS
int[][] dp = new int[m+1][n+1];
for (int i = m-1; i >= 0; i--)
    for (int j = n-1; j >= 0; j--)
        dp[i][j] = ...;
```

---

## How LCS differs from previous DP problems

| | Coin Change | House Robber | LCS |
|---|---|---|---|
| Parameters | 1 (amount) | 1 (index) | 2 (i, j) |
| Memo | 1D | 1D | 2D |
| Choices | variable (loop) | fixed (2) | fixed (match/skip) |
| Fill direction | L→R | R→L | R→L, bottom→top |
| Two inputs | no | no | yes — two strings |

---

## One Line to Remember

> If characters match — take 1 + LCS of rest of both strings. If not — skip one character from either string, take the max. Two parameters → 2D memo → fill bottom-right to top-left.
