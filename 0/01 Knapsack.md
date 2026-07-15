# 0/1 Knapsack — Notes

> **Problem:** Given weights and values of items and a knapsack capacity, find the maximum value you can carry. Each item picked at most once.
> Classic DP problem — no direct LeetCode link but pattern appears in LC 416, LC 474, LC 494

---

## How this relates to House Robber

House Robber — at each house, rob or skip (forced skip if adjacent).
Knapsack — at each item, take or skip (forced skip if too heavy).

Same include/exclude pattern. New dimension: remaining capacity.

---

## The choice at every item

```
At item i with remaining capacity W:

TAKE item i  → gain val[i], reduce capacity by wt[i]
               → valid only if wt[i] <= W
               → val[i] + solve(i+1, W-wt[i])

SKIP item i  → gain nothing, capacity unchanged
               → solve(i+1, W)

return max(take, skip)
```

---

## Version 1 — Plain Recursion

```java
private int solve(int W, int[] val, int[] wt, int index) {
    if (index >= val.length) return 0;   // no items left

    int take = 0;
    if (W >= wt[index])                  // only take if it fits
        take = val[index] + solve(W - wt[index], val, wt, index+1);

    int skip = solve(W, val, wt, index+1);

    return Math.max(take, skip);
}
```

---

## Version 2 — Memoization (Top-Down DP)

**Two things change:** `index` and `W` → `memo[index][W]`

```java
class Solution {
    int[][] memo;

    public int knapsack(int W, int[] val, int[] wt) {
        memo = new int[val.length+1][W+1];
        for (int[] arr : memo) Arrays.fill(arr, -1);
        return solve(W, val, wt, 0);
    }

    private int solve(int W, int[] val, int[] wt, int index) {
        if (index >= val.length) return 0;
        if (memo[index][W] != -1) return memo[index][W];  // cache hit

        int take = 0;
        if (W >= wt[index])
            take = val[index] + solve(W - wt[index], val, wt, index+1);

        int skip = solve(W, val, wt, index+1);

        return memo[index][W] = Math.max(take, skip);  // store and return
    }
}
```

---

## Version 3 — Tabulation (Bottom-Up DP)

**Fill direction:**
```
calls index+1  → needs index+1 filled → fill index RIGHT TO LEFT (n-1 down to 0)
calls W-wt[i]  → needs smaller W filled → fill W LEFT TO RIGHT (0 up to W)
```

```java
class Solution {
    public int knapsack(int W, int[] val, int[] wt) {
        int n = val.length;
        int[][] dp = new int[n+1][W+1];  // +1 for base case row

        for (int i = n-1; i >= 0; i--) {         // items right to left
            for (int j = 0; j <= W; j++) {        // capacity left to right
                int take = 0;
                if (j >= wt[i])
                    take = val[i] + dp[i+1][j-wt[i]];

                dp[i][j] = Math.max(take, dp[i+1][j]);  // skip = dp[i+1][j]
            }
        }

        return dp[0][W];  // answer at top-left
    }
}
```

Base case: `dp[n][*] = 0` handled automatically by array initialization.

---

## Slot Trace — weights=[1,3,4,5], values=[1,4,5,7], W=7

```
dp[4][*] = 0  (no items — base case)

i=3 (wt=5, val=7):
  j<5: take=0, dp[3][j]=max(0,0)=0
  j=5: take=7+dp[4][0]=7, dp[3][5]=7
  j=6: take=7+dp[4][1]=7, dp[3][6]=7
  j=7: take=7+dp[4][2]=7, dp[3][7]=7

i=2 (wt=4, val=5):
  j<4: dp[2][j]=0
  j=4: take=5+dp[3][0]=5, dp[2][4]=5
  j=5: take=5+dp[3][1]=5, skip=dp[3][5]=7, dp[2][5]=7
  j=6: take=5+dp[3][2]=5, skip=dp[3][6]=7, dp[2][6]=7
  j=7: take=5+dp[3][3]=5, skip=dp[3][7]=7, dp[2][7]=7

i=1 (wt=3, val=4):
  j=7: take=4+dp[2][4]=4+5=9, skip=dp[2][7]=7
  dp[1][7]=max(9,7)=9  ✅

i=0 (wt=1, val=1):
  j=7: take=1+dp[1][6], skip=dp[1][7]=9
  dp[0][7]=9

result = dp[0][7] = 9 ✅  (pick wt=3,val=4 AND wt=4,val=5 → 4+5=9)
```

---

## 5 Questions Applied

```
1. What changes between calls?
   → index (which item) AND W (remaining capacity)
   → memo[index][W], size [n+1][W+1]

2. Answer == return value?
   → YES — max value IS what we return
   → track inside, return directly

3. How many choices?
   → fixed: take (if fits) or skip
   → no loop needed

4. Base case meaning?
   → index==n → no items left → 0 value
   → W==0 → no capacity → 0 value (auto from initialization)

5. Fill direction?
   → calls index+1 → RIGHT TO LEFT for index
   → calls W-wt[i] → LEFT TO RIGHT for capacity
   → answer at dp[0][W]
```

---

## Knapsack vs LCS — 2D DP Comparison

| | LCS | Knapsack |
|---|---|---|
| Parameters | i (text1), j (text2) | index (item), W (capacity) |
| Fill i direction | right to left | right to left |
| Fill j direction | right to left | LEFT to right |
| Why j differs | needs j+1 (right) | needs j-wt[i] (left, smaller) |
| Base case | last row+col = 0 | last row = 0 |
| Answer | dp[0][0] | dp[0][W] |

**Key difference:** LCS fills both dimensions right to left. Knapsack fills capacity left to right because it needs SMALLER capacity (j-wt[i]) already computed.

---

## All DP Problems Solved — Pattern Map

```
1D DP — one changing parameter:
  Climbing Stairs  → fixed 2 choices (1step, 2step)     fill L→R
  House Robber     → fixed 2 choices (rob, skip)         fill R→L
  Coin Change      → variable choices (loop over coins)  fill L→R

2D DP — two changing parameters:
  LCS              → fixed choices (match/skip)          fill R→L, R→L
  Knapsack         → fixed choices (take/skip)           fill R→L, L→R
```

---

## One Line to Remember

> At each item: take it (gain value, reduce capacity) or skip it. Two parameters change (item index, capacity) → 2D memo. Fill items right to left, capacity left to right.
