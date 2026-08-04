# House Robber — Notes

> **Problem:** Rob houses in a line. Cannot rob two adjacent houses. Find maximum money.
> LC 198 — leetcode.com/problems/house-robber

---

## The choice at every house

At each house `i` you make exactly one decision:

```
ROB house i   → gain nums[i], skip i+1, next decision at i+2
SKIP house i  → gain 0,       next decision at i+1
```

---

## Step 1 — Plain Recursion

```java
int rob(int[] nums, int i) {
    if (i >= nums.length) return 0;   // no houses left

    int robCurrent  = nums[i] + rob(nums, i+2);  // rob this house
    int skipCurrent = rob(nums, i+1);             // skip this house

    return Math.max(robCurrent, skipCurrent);
}
```

**Recurrence:** `rob(i) = max(nums[i] + rob(i+2), rob(i+1))`

**Overlapping subproblems:** `rob(3)` called from `rob(1)` AND `rob(2)` — recomputed. Add memo.

---

## Step 2 — Memoization (Top-Down DP)

```java
class Solution {
    int[] memo;

    public int rob(int[] nums) {
        memo = new int[nums.length+1];
        Arrays.fill(memo, -1);
        return function(nums, 0);
    }

    private int function(int[] nums, int index) {
        if (index >= nums.length) return 0;
        if (memo[index] != -1) return memo[index];

        int robCurrent  = nums[index] + function(nums, index+2);
        int skipCurrent = function(nums, index+1);

        memo[index] = Math.max(robCurrent, skipCurrent);
        return memo[index];
    }
}
```

---

## Step 3 — Tabulation Right to Left (Bottom-Up DP)

**Why right to left:** recursion calls `rob(i+1)` and `rob(i+2)` — needs RIGHT neighbors.
RIGHT neighbors must be filled first → fill RIGHT TO LEFT.

```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n+2];   // +2 to avoid bounds check for dp[i+2]
        dp[n]   = 0;               // no houses left
        dp[n-1] = nums[n-1];       // last house — rob it

        for (int i = n-2; i >= 0; i--) {
            dp[i] = Math.max(nums[i] + dp[i+2], dp[i+1]);
        }
        return dp[0];
    }
}
```

**Trace for `[2,7,9,3,1]`:**
```
dp[5] = 0
dp[4] = 1
dp[3] = max(3+0,  1)  = 3
dp[2] = max(9+1,  3)  = 10
dp[1] = max(7+3,  10) = 10
dp[0] = max(2+10, 10) = 12 ✅
```

---

## Step 4 — Tabulation Left to Right (alternative)

**Different meaning of dp[i]:** max money from house 0 to house i (not i to end).

```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);   // can't rob both

        for (int i = 2; i < n; i++) {
            dp[i] = Math.max(nums[i] + dp[i-2], dp[i-1]);
        }
        return dp[n-1];
    }
}
```

**Trace for `[2,7,9,3,1]`:**
```
dp[0] = 2
dp[1] = max(2,7)    = 7
dp[2] = max(9+2, 7) = 11
dp[3] = max(3+7, 11)= 11
dp[4] = max(1+11,11)= 12 ✅
```

---

## Step 5 — Space Optimized O(1)

```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) return nums[0];

        int prev2 = nums[0];
        int prev1 = Math.max(nums[0], nums[1]);

        for (int i = 2; i < n; i++) {
            int curr = Math.max(nums[i] + prev2, prev1);
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

Only keep last two values — same sliding window as climbing stairs.

---

## Fill direction rule — how to know which way

```
Look at your recursion:

rob(i+1), rob(i+2)  →  i PLUS something  →  needs RIGHT neighbors
                     →  fill RIGHT TO LEFT ←←←
                     →  base cases at END, answer at dp[0]

climb(n-1), climb(n-2) → n MINUS something → needs LEFT neighbors
                       → fill LEFT TO RIGHT →→→
                       → base cases at START, answer at dp[n]
```

**The cheat code:**
> `f(i+something)` → fill right to left.
> `f(i-something)` → fill left to right.

---

## Side by side — right to left vs left to right

| | Right to Left | Left to Right |
|---|---|---|
| `dp[i]` means | max money from house i to END | max money from house 0 to house i |
| base cases | `dp[n]=0, dp[n-1]=nums[n-1]` | `dp[0]=nums[0], dp[1]=max(nums[0],nums[1])` |
| fill direction | `i = n-2 down to 0` | `i = 2 up to n-1` |
| answer | `dp[0]` | `dp[n-1]` |
| formula | `max(nums[i]+dp[i+2], dp[i+1])` | `max(nums[i]+dp[i-2], dp[i-1])` |

Both give same answer — different perspective on the same problem.

---

## All versions — complexity

| Version | Time | Space |
|---|---|---|
| Plain recursion | O(2^n) | O(n) stack |
| Memoization | O(n) | O(n) |
| Tabulation | O(n) | O(n) |
| Space optimized | O(n) | O(1) |

---

## My mistake — dp[1] without comparing dp[0]

```java
dp[1] = nums[1];                    // ❌ ignores nums[0]
dp[1] = Math.max(nums[0], nums[1]); // ✅ best of first two houses
```

In left-to-right, `dp[1]` means "best money from houses 0 and 1." You can't rob both — take the max. Without this, you lose the option of robbing house 0.

---

## One line to remember

> At each house: rob it (gain nums[i], skip next) OR skip it (move to next). Take the max. Recurrence: `dp[i] = max(nums[i] + dp[i+2], dp[i+1])`.
