# TC & SC Cheat Sheet

> Read this before calculating complexity for any problem.

---

## The 3-Question Checklist

```
Q1. How many unique subproblems?
    → multiply all parameter ranges

Q2. Is there a loop inside the recursion?
    → multiply by loop size

Q3. How deep is the stack?
    → count the parameter that drives recursion levels
    → compare with memo size, larger wins for SC
```

---

## Trick 1 — TC for Plain Recursion (no memo)

Count branching factor at each level:

```
1 choice per level   → O(n)        sum, reverse, countDigit
2 choices per level  → O(2^n)      knapsack, subsets, house robber
k choices per level  → O(k^n)      coin change → O(coins^amount)
n choices per level  → O(n!)       permutations
```

---

## Trick 2 — TC for Memoization

```
TC = unique subproblems × work per subproblem

unique subproblems:
  1 parameter  solve(i)      → n
  2 parameters solve(i, j)   → m × n
  2 parameters solve(i, W)   → n × W

work per subproblem:
  no loop inside  → O(1)
  loop of size k  → O(k)
```

---

## Trick 3 — SC for Memoization

```
SC = memo array size + stack depth

memo array size = same as unique subproblems
  1D memo → O(n)
  2D memo → O(n×W) or O(m×n)

stack depth = parameter that COUNTS THE LEVELS
  (not all parameters that change — only the one driving recursion depth)
  knapsack  → index drives depth → O(n)
  LCS       → i drives depth    → O(m)
  coin change → amount drives   → O(amount)

Total SC = larger of (memo, stack)
```

---

## Key insight — stack depth vs memo size

Even when TWO parameters change (like knapsack's `index` and `W`):
- `W` changes VALUE at each level but doesn't add more frames
- `index` counts the LEVELS → stack depth = n

```
Knapsack:
  memo = O(n×W)   ← 2D array
  stack = O(n)    ← index drives depth
  SC = O(n×W)     ← memo dominates
```

---

## Quick Reference Table

| Problem | Parameters | Unique subproblems | Loop? | Plain TC | Memo TC | Memo SC | Stack depth |
|---|---|---|---|---|---|---|---|
| Sum 1 to N | n | n | No | O(n) | — | O(n) | O(n) |
| Count Digit | n | log n | No | O(log n) | — | O(log n) | O(log n) |
| Tree height | nodes | n | No | O(n) | — | O(h) | O(h) |
| Climbing Stairs | n | n | No | O(2^n) | O(n) | O(n) | O(n) |
| House Robber | i | n | No | O(2^n) | O(n) | O(n) | O(n) |
| Coin Change | amount | amount | Yes (coins) | O(coins^amt) | O(amt×coins) | O(amt) | O(amt) |
| LCS | i, j | m×n | No | O(2^(m+n)) | O(m×n) | O(m×n) | O(m) |
| Knapsack | index, W | n×W | No | O(2^n) | O(n×W) | O(n×W) | O(n) |
| LIS | i | n | Yes (j<i) | O(2^n) | O(n²) | O(n) | O(n) |
| Edit Distance | i, j | m×n | No | O(3^(m+n)) | O(m×n) | O(m×n) | O(m) |
| Word Break | start | n | Yes (words) | O(2^n) | O(n×k) | O(n) | O(n) |
| Subsets | index | 2^n | No | O(2^n) | — | O(n) | O(n) |
| Permutations | index | n! | No | O(n!) | — | O(n) | O(n) |
| N-Queens | row | n! | No | O(n!) | — | O(n) | O(n) |
| Word Search | row,col,idx | m×n×L | No | O(m×n×4^L) | — | O(L) | O(L) |
| Reverse LL | n | n | No | O(n) | — | O(n) | O(n) |
| Merge Lists | m+n | m+n | No | O(m+n) | — | O(m+n) | O(m+n) |

---

## Tabulation Fill Direction — why i goes n-1 to 0 and j goes 0 to W

### Memoization vs Tabulation — opposite directions, same logic

```
MEMOIZATION                        TABULATION
─────────────────────────────────  ──────────────────────────────────
start: solve(0, W)                 start: fill dp[n][*] = 0 (base case)
recurse DOWN: index 0 → n          fill UP: i from n-1 down to 0
W shrinks as items taken           j grows: 0 up to W
base case at bottom (index=n)      base case already at bottom (row n)
answer: solve(0, W)                answer: dp[0][W]
```

### Why i goes from n-1 DOWN to 0

Base case is `dp[n][*] = 0` — no items left.
To compute `dp[0][W]` you need `dp[1][*]` filled first.
To fill `dp[1][*]` you need `dp[2][*]` filled first.
→ must fill FROM base case UPWARD → i from n-1 down to 0.

### Why j goes from 0 UP to W

```java
dp[i][j] = max(val[i] + dp[i+1][j - wt[i]], dp[i+1][j])
```

`dp[i][j]` needs `dp[i+1][j - wt[i]]` — a SMALLER j value.
Smaller j must be filled before larger j → fill j from 0 up to W.

### The universal fill direction rule

```
dp[i][j] needs dp[i+1][...] → i+1 must exist → fill i from n-1 DOWN to 0
dp[i][j] needs dp[i][j-1]   → smaller j needed → fill j from 0 UP to W
dp[i][j] needs dp[i-1][...]  → i-1 must exist → fill i from 1 UP to n
dp[i][j] needs dp[i][j+1]   → larger j needed → fill j from W DOWN to 0
```

One line: **fill in the direction that ensures what you need is already computed.**

---

## Tabulation SC — always drop the stack

```
Tabulation SC = memo array only (no recursion stack)
  1D DP → O(n)
  2D DP → O(n×W) or O(m×n)

Space optimized tabulation:
  only keep last row/col → O(n) or O(W)
  only keep 2 variables  → O(1)  (climbing stairs, house robber)
```

---

## Memoization vs Tabulation — which is better?

| | Memoization | Tabulation |
|---|---|---|
| Direction | Top-down | Bottom-up |
| Recursion stack | Yes → O(n) extra | No |
| Stack overflow risk | Yes for large n | No |
| Computes only needed subproblems | Yes | No — fills entire table |
| Code readability | Easier — follows recursion | Requires knowing fill direction |
| Space optimization | Hard | Easy — keep last row/2 variables |
| Cache friendly | No | Yes — sequential access |

### When memoization wins
```
Sparse subproblems — not all combinations get called
→ only computes what's needed
→ tabulation fills everything regardless
```

### When tabulation wins
```
1. No stack overflow — safe for large inputs
2. Space optimization easy:
   knapsack     → keep 2 rows   → O(W) instead of O(n×W)
   house robber → keep 2 vars   → O(1) instead of O(n)
3. Slightly faster — no function call overhead
4. Cache friendly — sequential array access
```

### Interview strategy
```
Step 1 → write memoization first (easier, follows recursion)
Step 2 → mention tabulation ("can convert to bottom-up")
Step 3 → mention space opt ("can reduce SC further to O(W) or O(1)")
```

**One line:** Memoization is easier to write. Tabulation is better in practice — no stack, space optimizable, no overflow risk.

---

## Big-O Rules — drop constants

```
O(n/2)   = O(n)
O(2n)    = O(n)
O(n + h) = O(n)  if h ≤ n
O(n×W + n) = O(n×W)  ← larger dominates
```

---

## Shrink pattern → TC pattern

```
subtract 1     (n-1)    → O(n)
divide by 2    (n/2)    → O(log n)
divide by 10   (n/10)   → O(log n)
two branches   (n-1, n-2) → O(2^n) without memo
loop inside              → multiply by loop size
```
