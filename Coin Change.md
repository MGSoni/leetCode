# Coin Change — Notes

> **Problem:** Given coins and a target amount, find minimum number of coins to make the amount. Return -1 if impossible.
> LC 322 — leetcode.com/problems/coin-change

---

## The Pattern — Recursion with Loop → DP

At every amount, try every coin. Pick the one that gives minimum result.

```
solve(amount) = 1 + min(solve(amount - coin)) for every coin
```

This is NOT backtracking — you need ONE answer (minimum), not ALL solutions.
Same subproblems repeat → add memo → DP.

---

## Version 1 — Plain Recursion

```java
int solve(int[] coins, int amount) {
    if (amount == 0) return 0;    // arrived — 0 coins needed
    if (amount < 0)  return -1;   // overshot — impossible

    int min = Integer.MAX_VALUE;  // no valid path found yet

    for (int coin : coins) {
        int result = solve(coins, amount - coin);
        if (result != -1) {
            min = Math.min(min, 1 + result);  // 1 coin used + rest
        }
    }

    return min == Integer.MAX_VALUE ? -1 : min;
}
```

---

## Version 2 — Memoization (Top-Down DP)

**Three additions:**
```
1. memo array sized amount+1, filled with -2
2. check memo before computing
3. store in memo before returning
```

```java
class Solution {
    int[] memo;

    public int coinChange(int[] coins, int amount) {
        memo = new int[amount + 1];
        Arrays.fill(memo, -2);           // -2 = not computed yet
        return solve(coins, amount);
    }

    private int solve(int[] coins, int amount) {
        if (amount == 0) return 0;
        if (amount < 0)  return -1;
        if (memo[amount] != -2) return memo[amount];  // cache hit

        int min = Integer.MAX_VALUE;
        for (int coin : coins) {
            int result = solve(coins, amount - coin);
            if (result != -1) {
                min = Math.min(min, 1 + result);
            }
        }

        memo[amount] = (min == Integer.MAX_VALUE) ? -1 : min;
        return memo[amount];
    }
}
```

---

## Version 3 — Tabulation (Bottom-Up DP)

**The conversion rule:**
```
recursive function argument = amount   → OUTER loop: for i = 1 to amount
loop inside recursive function = coins → INNER loop: for coin in coins
```

**Fill direction:** `solve(amount - coin)` → needs LEFT → fill LEFT TO RIGHT.

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);  // amount+1 = "infinity"
        dp[0] = 0;                    // base case

        for (int i = 1; i <= amount; i++) {      // outer — every amount
            for (int coin : coins) {             // inner — try every coin
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], 1 + dp[i - coin]);
                }
            }
        }

        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

**Why `amount+1` as infinity not `Integer.MAX_VALUE`:**
`1 + Integer.MAX_VALUE` overflows to negative — wrong answer.
`amount+1` is safely larger than any valid answer, no overflow risk.

---

## Slot Trace — coins=[1,2,5], amount=5

```
dp = [0, 6, 6, 6, 6, 6]   ← amount+1=6 as infinity

i=1: coin=1: dp[1]=min(6,1+dp[0])=1
     dp=[0,1,6,6,6,6]

i=2: coin=1: dp[2]=min(6,1+dp[1])=2
     coin=2: dp[2]=min(2,1+dp[0])=1
     dp=[0,1,1,6,6,6]

i=3: coin=1: dp[3]=min(6,1+dp[2])=2
     coin=2: dp[3]=min(2,1+dp[1])=2
     dp=[0,1,1,2,6,6]

i=4: coin=1: dp[4]=min(6,1+dp[3])=3
     coin=2: dp[4]=min(3,1+dp[2])=2
     dp=[0,1,1,2,2,6]

i=5: coin=1: dp[5]=min(6,1+dp[4])=3
     coin=2: dp[5]=min(3,1+dp[3])=3
     coin=5: dp[5]=min(3,1+dp[0])=1
     dp=[0,1,1,2,2,1]

dp[5]=1 ✅ (one coin of value 5)
```

---

## Key Decisions — The 5 Questions Applied

```
1. What changes between calls?
   → only amount changes, coins always fully available
   → memo[amount] only, size = amount+1

2. Answer == return value?
   → YES (min coins for this amount IS what we return)
   → min lives INSIDE function, returned directly

3. How many choices?
   → variable (as many as coins.length)
   → loop over coins

4. Base case meaning?
   → amount==0 → arrived → 0 coins needed → return 0
   → amount<0  → overshot → impossible → return -1

5. Fill direction?
   → solve(amount-coin) → f(i-x) → LEFT TO RIGHT
   → base case at START (dp[0]=0), answer at END (dp[amount])
```

---

## Why No Index Parameter

Combinations needed index because: once you pick number 3, you can't go back to 1 or 2.

Coin change: you can use any coin unlimited times, in any order. No restriction.

```
coins=[1,2,5], amount=11
→ use 5,5,1   ✅
→ use 1,5,5   ✅  (same coins, different order — both valid)
→ use 5 twice ✅  (unlimited reuse)
```

The loop already tries every coin at every call. Adding index would wrongly restrict which coins are available.

---

## Why `min` Lives Inside Not Outside

```
min inside  → "best coin choice at THIS amount" → returned to caller
            → use when answer == return value

external    → "best seen anywhere across all calls" → never returned
variable      → use when answer != return value (diameter, subsets)
```

**What breaks with global min:**
```
solve(1) sets globalMin=1 (min coins for amount 1)
solve(3) reads globalMin=1 thinking it's min for amount 3
→ returns 1 instead of 2  ❌
```

Global min mixes answers from different amounts — each amount needs its own private minimum.

---

## Why Sentinel is `-2` not `-1`

```
-1 = valid answer meaning "impossible"
-2 = sentinel meaning "not computed yet"
```

If you use `-1` for both — can't tell if `memo[5]=-1` means "impossible" or "not computed."
Always pick a sentinel that can never be a real answer.

---

## Tabulation Conversion Template

```
MEMOIZATION                           TABULATION
─────────────────────────────────     ──────────────────────────────
solve(changing_param):                for changing_param in range:
    base cases                            (handle base before loop)
    check memo
    for choice in choices:                for choice in choices:
        sub = solve(next_state)               dp[i] = combine(dp[prev])
        result = combine(sub)
    store in memo
    return result                     return dp[target]

outer loop = recursive function argument
inner loop = loop inside recursive function
```

---

## My Mistakes

### Mistake 1 — Used backtracking instead of DP
Saw "try all combinations" → reached for backtracking. Needed ONE answer (minimum) not ALL solutions → DP.

### Mistake 2 — Added index parameter
```java
solve(coins, amount, index)  // ❌ restricts coin choices unnecessarily
solve(coins, amount)         // ✅ all coins always available
```

### Mistake 3 — Wrong memo size
```java
memo = new int[coins.length+1];  // ❌ caching by coin not by amount
memo = new int[amount+1];        // ✅ one slot per possible amount
```

### Mistake 4 — Wrong loop order in tabulation
```java
for (int i = 0; i < coins.length; i++)  // ❌ outer loop over coins
for (int i = 1; i <= amount; i++)        // ✅ outer loop over amounts
```

Outer loop = recursive function argument = amount.
Inner loop = loop inside recursive function = coins.

---

## All Three Versions — Complexity

| Version | Time | Space |
|---|---|---|
| Plain recursion | O(coins^amount) | O(amount) stack |
| Memoization | O(amount × coins) | O(amount) |
| Tabulation | O(amount × coins) | O(amount) |

---

## One Line to Remember

> For each amount, try every coin, take minimum valid result. Outer loop = amounts, inner loop = coins. Cache by amount only — coins always fully available.
