# Longest Increasing Subsequence — Notes

> **Problem:** Given an integer array, find the length of the longest strictly increasing subsequence.
> LC 300 — leetcode.com/problems/longest-increasing-subsequence

---

## Why this problem is hard

Four things confused me simultaneously:
1. Not being able to come up with the recursion
2. Confusing the SUBJECT and HELPER inside the loop
3. Storing result in wrong memo slot
4. Returning `dp[n-1]` instead of global max
5. Not understanding why tabulation prefills with 1

This note answers all five clearly.

---

## Step 1 — Build intuition by hand first

Given `[2, 5, 3, 7]`, for each position ask:
**"What is the longest increasing chain that ENDS HERE?"**

```
Position 0, value=2:
  No previous elements.
  Chain = [2] alone.
  Length = 1

Position 1, value=5:
  Look back at position 0: 2 < 5? YES → extend [2] → [2,5]
  Length = 2

Position 2, value=3:
  Look back at position 0: 2 < 3? YES → extend [2] → [2,3], length 2
  Look back at position 1: 5 < 3? NO  → can't extend [2,5]
  Length = 2

Position 3, value=7:
  Look back at position 0: 2 < 7? YES → extend [2],   length 2
  Look back at position 1: 5 < 7? YES → extend [2,5], length 3
  Look back at position 2: 3 < 7? YES → extend [2,3], length 3
  Length = 3

Answers: [1, 2, 2, 3]
Global answer = max(1,2,2,3) = 3
```

**This hand computation IS the algorithm.** Code just automates it.

---

## Step 2 — Why you need a loop inside recursion

At position `i`, you don't know which previous position gives the best chain to extend — so you try ALL of them and pick the best.

```
Fixed choices (2-3)    → write each call explicitly (house robber, climbing stairs)
Variable choices       → loop over choices (coin change, LIS)
```

LIS loop purpose: search all previous positions for valid predecessors.
Coin change loop purpose: try all coins at current amount.

Different purpose, same mechanism — a loop inside recursion.

---

## Step 3 — The three roles inside solve(index)

```
index  = SUBJECT    — who we're computing for
j      = HELPER     — who we ask for information
best   = ACCUMULATOR — collects the best answer for SUBJECT
```

```java
int solve(int[] nums, int index) {
    int best = 1;                           // ACCUMULATOR — starts at 1

    for (int j = 0; j < index; j++) {      // loop over HELPERs
        if (nums[j] < nums[index]) {        // can HELPER j be extended?
            int jAnswer = solve(nums, j);   // ask HELPER for its answer
            best = Math.max(best, 1 + jAnswer);  // update ACCUMULATOR
        }
    }

    memo[index] = best;   // store ACCUMULATOR in SUBJECT's memo slot
    return memo[index];   // return SUBJECT's answer
}
```

**The rule:** inside `solve(index)`, you ONLY write to `memo[index]`.
`j` is just a tool — never store anything in `memo[j]` from inside `solve(index)`.

---

## Step 4 — Why `best = 1` not `best = 0`

Every element alone is a valid subsequence of length 1. Even if NO previous element is smaller than `nums[i]`, you can always start a new chain with just `nums[i]`.

```
[7, 7, 7, 7] → no j satisfies nums[j] < nums[i] (equal, not strictly less)
best stays 1 for every position
answer = 1 ✅
```

If you used `best = 0`, you'd return 0 for elements with no valid predecessor — wrong.

---

## Step 5 — Why the outer loop in main function

`solve(i)` only tells you LIS ending at position `i`. The global LIS might end anywhere.

**The case that breaks `return dp[n-1]`:**

```
nums = [1, 3, 5, 2]

dp[0]=1  ([1])
dp[1]=2  ([1,3])
dp[2]=3  ([1,3,5])   ← LIS ends HERE
dp[3]=2  ([1,2])

dp[n-1] = dp[3] = 2   ❌  wrong — answer is 3
global max = max(1,2,3,2) = 3  ✅
```

LIS ends at position 2, not position 3. Returning `dp[n-1]` gives the wrong answer.

**Always track max across ALL positions:**

```java
int max = 1;
for (int i = 0; i < n; i++) {
    max = Math.max(max, solve(nums, i));
}
return max;
```

---

## Version 1 — Plain Recursion

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int max = 0;
        for (int i = 0; i < nums.length; i++) {
            max = Math.max(max, solve(nums, i));
        }
        return max;
    }

    private int solve(int[] nums, int index) {
        int best = 1;                            // element alone = length 1

        for (int j = 0; j < index; j++) {       // look at all predecessors
            if (nums[j] < nums[index]) {         // strictly increasing?
                best = Math.max(best, 1 + solve(nums, j));  // extend chain
            }
        }

        return best;
    }
}
```

---

## Version 2 — Memoization (Top-Down DP)

**What changes between calls?** Only `index` → `memo[index]`, size `n`.

```java
class Solution {
    int[] memo;

    public int lengthOfLIS(int[] nums) {
        memo = new int[nums.length];
        Arrays.fill(memo, -1);                   // -1 = not computed yet

        int max = 0;
        for (int i = 0; i < nums.length; i++) {
            max = Math.max(max, solve(nums, i));  // try every ending position
        }
        return max;
    }

    private int solve(int[] nums, int index) {
        if (memo[index] != -1) return memo[index];  // cache hit

        int best = 1;                               // SUBJECT's accumulator
        for (int j = 0; j < index; j++) {          // loop over HELPERs
            if (nums[j] < nums[index]) {
                best = Math.max(best, 1 + solve(nums, j));  // use HELPER's answer
            }
        }

        return memo[index] = best;                  // store in SUBJECT's slot
    }
}
```

---

## Version 3 — Tabulation (Bottom-Up DP)

**Fill direction:** `solve(j)` where `j < index` → needs LEFT → fill LEFT TO RIGHT.

**Prefill with 1** because every element alone = length 1 (the base case value).

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);          // base case — every element alone = 1

        int max = 1;
        for (int i = 1; i < n; i++) {              // SUBJECT — outer loop
            for (int j = 0; j < i; j++) {          // HELPER — inner loop
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], 1 + dp[j]);  // extend HELPER's chain
                }
            }
            max = Math.max(max, dp[i]);             // update global max
        }
        return max;
    }
}
```

---

## The confusion — how `best` becomes `dp[i]` in tabulation

This was the biggest confusion. Here's the direct mapping:

```
MEMOIZATION                          TABULATION
───────────────────────────────────  ───────────────────────────────────
solve(index):                        for i from 0 to n-1:

  int best = 1                         dp[i] = 1  (prefilled)

  for j from 0 to index-1:            for j from 0 to i-1:
    if nums[j] < nums[index]:            if nums[j] < nums[i]:
      best = max(best,                     dp[i] = max(dp[i],
               1 + solve(j))                        1 + dp[j])

  memo[index] = best                   (dp[i] already updated directly)
  return memo[index]                   max = max(max, dp[i])
```

| Memoization | Tabulation | Meaning |
|---|---|---|
| `solve(index)` | `i` in outer loop | SUBJECT — who we compute for |
| `memo[index]` | `dp[i]` | stored answer for subject |
| `int best = 1` | `Arrays.fill(dp, 1)` | starting value before any j helps |
| `j` in loop | `j` in inner loop | HELPER — previous positions |
| `solve(j)` | `dp[j]` | helper's already-computed answer |
| `1 + solve(j)` | `1 + dp[j]` | extend helper's chain by 1 |
| `memo[index] = best` | `dp[i]` updated in loop | store subject's final answer |

**The only real difference:**
In memoization, `best` accumulates inside the function then gets stored in `memo[index]` at the end.
In tabulation, `dp[i]` plays both roles — starts at 1 (prefill) and gets updated directly.
`dp[i]` in tabulation IS `memo[index]` in memoization. Same thing.

**j is still the HELPER in tabulation:**
```java
dp[j]  = helper's answer (computed in earlier iteration)
dp[i]  = subject's answer (being built right now)
```

Roles never changed — just variable names shifted slightly.

---

## Slot Trace — `[2, 5, 3, 7]`

```
dp = [1, 1, 1, 1]   ← prefilled

i=1, nums[1]=5:
  j=0: nums[0]=2 < 5 → dp[1]=max(1, 1+dp[0])=max(1,2)=2
  max=2
  dp=[1,2,1,1]

i=2, nums[2]=3:
  j=0: nums[0]=2 < 3 → dp[2]=max(1, 1+dp[0])=max(1,2)=2
  j=1: nums[1]=5 < 3? NO → skip
  max=2
  dp=[1,2,2,1]

i=3, nums[3]=7:
  j=0: nums[0]=2 < 7 → dp[3]=max(1, 1+dp[0])=max(1,2)=2
  j=1: nums[1]=5 < 7 → dp[3]=max(2, 1+dp[1])=max(2,3)=3
  j=2: nums[2]=3 < 7 → dp[3]=max(3, 1+dp[2])=max(3,3)=3
  max=3
  dp=[1,2,2,3]

return max=3 ✅
```

---

## My Mistakes

### Mistake 1 — Storing in wrong memo slot

```java
// ❌ storing in j (helper) not index (subject)
memo[j] = Math.max(memo[j], 1 + solve(nums, j));

// ✅ accumulate in best, store in memo[index]
best = Math.max(best, 1 + solve(nums, j));
memo[index] = best;
```

**Root cause:** confused who the subject was. Inside `solve(index)`, ONLY write to `memo[index]`.

### Mistake 2 — Returning `dp[n-1]`

```java
return dp[n-1];  // ❌ LIS might not end at last position
```

```java
// ✅ track max across all positions
int max = 1;
for (int i = 0; i < n; i++) max = Math.max(max, dp[i]);
return max;
```

### Mistake 3 — Missing `nums[j] < nums[i]` check

```java
best = Math.max(best, 1 + dp[j]);          // ❌ extends any chain
if (nums[j] < nums[i])                      // ✅ only extend if increasing
    best = Math.max(best, 1 + dp[j]);
```

### Mistake 4 — Initializing dp with -1 not 1

```java
Arrays.fill(dp, -1);  // ❌ base case is 1, not -1
Arrays.fill(dp, 1);   // ✅ every element alone = length 1
```

### Mistake 5 — Setting memo before calling solve

```java
memo[i] = Math.max(memo[i], solve(nums, i));  // ❌ sets memo before solve runs
                                               // solve sees memo[i]!=-1, returns -1 immediately
max = Math.max(max, solve(nums, i));           // ✅ let solve handle memo internally
```

---

## 5 Questions Applied

```
1. What changes between calls?
   → only index → memo[index], size n

2. Answer == return value?
   → NO — solve(i) returns LIS ending at i
          final answer = max across all i
   → need outer loop in main, track max separately

3. How many choices?
   → variable → loop over predecessors j

4. Base case meaning?
   → every element alone = length 1
   → best=1 in recursion, Arrays.fill(dp,1) in tabulation
   → no explicit base case needed — loop just doesn't run for i=0

5. Fill direction?
   → solve(j) where j<i → needs LEFT → fill LEFT TO RIGHT
   → prefill with 1 (base case value)
   → answer = max across all dp[i]
```

---

## Why this differs from other DP problems

| Problem | solve(i) returns | Final answer |
|---|---|---|
| House Robber | max money from i to end | solve(0) directly |
| Coin Change | min coins for amount | solve(amount) directly |
| Climbing Stairs | ways to reach n | solve(n) directly |
| LIS | LIS ending at i | max(solve(0)...solve(n-1)) |

LIS is the only one where the return value ≠ final answer. That's why it needs an outer loop to collect the global maximum — unlike the others where calling solve once on the right input gives you the answer directly.

---

## One Line to Remember

> `solve(i)` = longest chain ending at i. Look backward at all j where nums[j]<nums[i], extend the best chain. Final answer = max across ALL positions, not just the last one.
