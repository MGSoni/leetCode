# Climbing Stairs — Notes

> **Problem:** You can climb 1 or 2 steps at a time. How many distinct ways to reach the top of n stairs?
> LC 70 — leetcode.com/problems/climbing-stairs

---

## The core insight — every DP problem is recursion with memory

```
Recursion     → correct, slow     O(2^n)
Memoization   → correct, fast     O(n) time, O(n) space  (top-down DP)
Tabulation    → correct, fastest  O(n) time, O(1) space  (bottom-up DP)
```

Always start with recursion. Add memo. Then optionally convert to tabulation.

---

## Step 1 — Plain Recursion

**3-question recipe:**

```
Base case:  n==0 → 1 way (already at top — count this path)
            n<0  → 0 ways (overshot — invalid path)
Shrink:     from stair n, came from n-1 (1 step) or n-2 (2 steps)
Combine:    ways(n) = ways(n-1) + ways(n-2)
```

```java
int climb(int n) {
    if (n < 0)  return 0;   // overshot
    if (n == 0) return 1;   // arrived — 1 valid path
    return climb(n-1) + climb(n-2);
}
```

**Why `n==0` returns 1 not 0:**
`n==0` means you've arrived at the top — this is 1 valid path, not 0.
`climb(0)` represents "took a 2-step jump and landed exactly at the top" — that's 1 way.

```
reached goal exactly → return 1  (one valid path)
overshot/invalid     → return 0  (dead end)
```

**The overlap problem — why it's slow:**
```
climb(5)
  climb(4)
    climb(3) ←──┐
      climb(2)  │
      climb(1)  │
    climb(3) ───┘  ← recomputed!
    climb(2) ──────┐
  climb(3) ────────┘  ← recomputed again!
```

`climb(3)` computed 3 times, `climb(2)` computed 5 times → O(2^n).

---

## Step 2 — Memoization (Top-Down DP)

Same recursion. Three additions:

```
1. create memo array filled with -1 (means "not computed yet")
2. check memo BEFORE computing
3. store result in memo BEFORE returning
```

```java
class Solution {
    int[] memo;

    public int climbStairs(int n) {
        memo = new int[n+1];
        Arrays.fill(memo, -1);
        return function(n);
    }

    private int function(int n) {
        if (n < 0)  return 0;
        if (n == 0) return 1;
        if (memo[n] != -1) return memo[n];        // already computed — return instantly

        memo[n] = function(n-1) + function(n-2);  // compute and store
        return memo[n];
    }
}
```

**Trace for n=5 with memo:**
```
function(5)
  function(4)
    function(3)
      function(2)
        function(1) = function(0)+function(-1) = 1+0 = 1, memo[1]=1
        function(0) → 1 (memo hit)
        memo[2]=2, return 2
      function(1) → 1  ← memo hit ✅ no recomputation
      memo[3]=3, return 3
    function(2) → 2    ← memo hit ✅
    memo[4]=5, return 5
  function(3) → 3      ← memo hit ✅
  memo[5]=8, return 8

result = 8 ✅ every subproblem computed exactly once
```

---

## Step 3 — Tabulation (Bottom-Up DP)

Build from bottom up instead of top down. No recursion — fill a table iteratively.

```java
class Solution {
    public int climbStairs(int n) {
        if (n == 1) return 1;
        if (n == 2) return 2;

        int prev1 = 1;   // dp[1] = 1 way
        int prev2 = 2;   // dp[2] = 2 ways
        int result = 0;

        for (int i = 3; i <= n; i++) {
            result = prev1 + prev2;  // dp[i] = dp[i-1] + dp[i-2]
            prev1 = prev2;           // slide window forward
            prev2 = result;
        }

        return result;
    }
}
```

Only two variables instead of an array — O(1) space. Optimal.

**Why only two variables:**
`dp[i]` only depends on the previous two values. No need to store the whole array — just keep a sliding window of size 2.

---

## All three versions side by side

```java
// VERSION 1 — plain recursion O(2^n) time, O(n) stack space
int climb(int n) {
    if (n < 0)  return 0;
    if (n == 0) return 1;
    return climb(n-1) + climb(n-2);
}

// VERSION 2 — memoization O(n) time, O(n) space (top-down DP)
int climb(int n) {
    if (n < 0)  return 0;
    if (n == 0) return 1;
    if (memo[n] != -1) return memo[n];
    memo[n] = climb(n-1) + climb(n-2);
    return memo[n];
}

// VERSION 3 — tabulation O(n) time, O(1) space (bottom-up DP)
int prev1=1, prev2=2, result=0;
for (int i = 3; i <= n; i++) {
    result = prev1 + prev2;
    prev1 = prev2;
    prev2 = result;
}
return result;
```

---

## My mistake — wrong base case

```java
if (n == 0) return 0;  // ❌ "no steps remaining = 0 ways"
if (n == 0) return 1;  // ✅ "already at top = 1 valid path"
```

**Root cause:** confused "number of steps remaining" with "number of valid paths from here."
`n==0` doesn't mean "no steps" — it means "you've arrived." Arrival = 1 valid path.

---

## The DP pattern — three additions to any recursion

```
recursion + these three lines = memoization (DP):

int[] memo = new int[n+1];          // 1. create cache
Arrays.fill(memo, -1);

if (memo[n] != -1) return memo[n];  // 2. check cache before computing

memo[n] = function(n-1) + ...;      // 3. store before returning
return memo[n];
```

This exact pattern applies to every DP problem. Only the recurrence relation changes.

---

## Recurrence relation

```
climb(n) = climb(n-1) + climb(n-2)
```

This is identical to Fibonacci. Climbing stairs IS Fibonacci — just with different base cases.

---

## One line to remember

> Ways to reach stair n = ways from stair n-1 + ways from stair n-2. Base case: n==0 returns 1 (arrived), n<0 returns 0 (overshot).
