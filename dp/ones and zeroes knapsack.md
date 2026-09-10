# Ones and Zeroes (LeetCode 474) — Multi-Constraint 0/1 Knapsack

*(Stand-in for LOKAL's "modified Knapsack with altered constraints" question — closest well-known variant, trains the same "add a dimension to the DP state" skill.)*

## Why DP, not Greedy (knapsack intuition)
Counter-example showing greedy fails:
```
Capacity W = 10
Item A: weight 6, value 10
Item B: weight 5, value 6
Item C: weight 5, value 6
```
- Greedy (highest value first) picks A → leftover capacity 4, unusable → total value = 10
- Optimal picks B + C → weight 10 fits exactly → total value = 12

**Lesson:** a locally "best" pick can waste leftover capacity that a different combination would have used better. Knapsack needs DP to explore combinations, not commit greedily.

## Problem
Given binary strings `strs[]` and integers `m` (max zeros), `n` (max ones), find the size of the largest subset where total zeros ≤ `m` and total ones ≤ `n`. Each string usable at most once.

## Why this is knapsack
Each string = an item with two "costs" (its zero-count and one-count) instead of one weight. Two capacities (`m`, `n`) instead of one. Still 0/1 (each item used once) → extend the classic 1D knapsack DP by one dimension.

## Classic 0/1 Knapsack recap (base pattern)
`dp[i][w]` = max value using first `i` items with capacity `w`.
```
dp[i][w] = max( dp[i-1][w],  val[i] + dp[i-1][w - wt[i]] )   // second term only if wt[i] <= w
```
Base case: `dp[0][w] = 0`, `dp[i][0] = 0`.

## Recurrence (extended to two constraints)
State: `dp[index][zerosLeft][onesLeft]` = max subset size using first `index` strings with `zerosLeft`/`onesLeft` budget remaining.

```
dp[index][zerosLeft][onesLeft] = max(
    dp[index-1][zerosLeft][onesLeft],                                     // don't take
    1 + dp[index-1][zerosLeft - zeroCount][onesLeft - oneCount]           // take, if both fit
)
```
Base case: `dp[0][*][*] = 0`.

**Must check both constraints before taking an item** — checking only one risks silently overflowing the other.

## Java Implementation (top-down, memoized)
```java
class Solution {
    int[][][] dp;
    int[] zeroCount, oneCount;

    public int findMaxForm(String[] strs, int m, int n) {
        dp = new int[strs.length + 1][m + 1][n + 1];
        for (int[][] twoD : dp) {
            for (int[] oneD : twoD) {
                Arrays.fill(oneD, -1);
            }
        }

        // precompute zero/one counts once — avoids recounting per state
        zeroCount = new int[strs.length];
        oneCount = new int[strs.length];
        for (int i = 0; i < strs.length; i++) {
            for (char c : strs[i].toCharArray()) {
                if (c == '0') zeroCount[i]++;
                else oneCount[i]++;
            }
        }

        return function(strs, m, n, 0);
    }

    private int function(String[] strs, int m, int n, int index) {
        if (index >= strs.length) return 0;
        if (dp[index][m][n] != -1) return dp[index][m][n];

        int zeros = zeroCount[index];
        int ones = oneCount[index];

        int take = 0;
        if (zeros <= m && ones <= n) {
            take = 1 + function(strs, m - zeros, n - ones, index + 1);
        }
        int notTake = function(strs, m, n, index + 1);

        dp[index][m][n] = Math.max(take, notTake);
        return dp[index][m][n];
    }
}
```

## Bugs I hit (watch for these again)
- Swapped comparisons: checked `ones <= m` and `zeros <= n` when `m` = zero-budget and `n` = one-budget. Always double-check which parameter maps to which constraint.
- Recomputed zero/one counts inside `function()` on every call — wasteful since a string's counts don't depend on `m`/`n`. Fixed by precomputing once into `zeroCount[]`/`oneCount[]` arrays.
- Tried to reuse the `dp` array to store precomputed counts — wrong because `dp` is indexed by `(index, m, n)` while counts only depend on `index`; would overwrite real memoized answers.
- 3D array sizing: `index` ranges `0` to `strs.length` inclusive (base case is `index >= strs.length`), so array needs `strs.length + 1` in the first dimension, not `strs.length`.

## Complexity
Let `L = strs.length`, `K = max string length`.

| Step | Cost |
|---|---|
| Precompute zero/one counts | O(L·K) |
| Init `dp` to -1 | O(L·m·n) |
| Recursion (distinct states × O(1) work each) | O(L·m·n) |
| **Total time** | **O(L·K + L·m·n)** |
| `dp` array space | O(L·m·n) |
| Recursion stack depth | O(L) |
| **Total space** | **O(L·m·n + L)** ≈ O(L·m·n) |

## General pattern
"At most K items" / multi-constraint selection problems → extend classic knapsack DP by adding one dimension per constraint.

## Practice link
LeetCode 474: https://leetcode.com/problems/ones-and-zeroes/
