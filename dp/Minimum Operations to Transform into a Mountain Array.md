# Minimum Operations to Transform into a Mountain Array (DP with running minimum)

**Source:** LeetCode Bi-weekly Contest 196 (problem number and link not confirmed). Contest-hard medium. Not a priority for interviews, so treat it as a "revisit if time" item.

**Problem:** Given `nums` (length n), one operation changes any element by +1 or -1, and all elements must stay at least 1. A mountain array has a peak index `p` with `0 < p < n-1`, strictly increasing up to `p` and strictly decreasing after it. Return the minimum operations to make `nums` a mountain, or `-1` if impossible (only when n < 3).

**Pattern:** "Choose the cheapest target array." DP over (index, final value), with a running minimum for "cheapest cell strictly to the left." Build one table from the left, one from the right (reversed array), and combine at each peak.

---

## Core insight

- Pick a target array `t`. Cost = sum of `|nums[i] - t[i]|`. The real job is choosing the cheapest valid `t`.
- Raising only the biggest middle element is **not enough**. `[1,1,1,1,1]` needs `[1,2,3,2,1]`, costing 4.
- The peak's value is a real choice. `[9,4,9]`: keeping the peak at 4 costs 12, raising it to 10 costs 6.
- Neighbours' choices clash (strictly increasing), so choices are linked. That is the signal for DP.

## The left table

`L[i][v]` = cheapest cost to make indexes 0 to i strictly increasing, with index i ending at value v.

```
L[0][v] = |nums[0] - v|
L[i][v] = |nums[i] - v| + (cheapest cell in row i-1, strictly LEFT of column v)
```

Example: left side `5, 1, 9`, peak 8.

```
final value →     1    2    3    4    5    6    7
idx 0 (orig 5)    4    3    2    1    0    .    .
idx 1 (orig 1)    .    5    5    5    5    5    .
idx 2 (orig 9)    .    .   11   10    9    8    7
```

A dot is `INF`: that value is impossible there (index i needs a value of at least i+1, since all values are at least 1).

## Running minimum (no rescanning)

```
row above:      1   0   1   2   3   4
best so far:   INF  1   0   0   0   0     <- cheapest strictly left of each column
```

At each column `v`, in this order:
1. compute the cell: `|nums[i] - v| + best` (stay `INF` if `best` is `INF`)
2. then `best = min(best, rowAbove[v])`

If you update `best` first, column `v` would feed its own cell, but it must be strictly left.

Trace on `[2, 2, 3]`, row 1: `v=1` gives INF, then best = 1. `v=2` gives 0 + 1 = 1, then best = 0. `v=3` gives 1 + 0 = 1, then best = 0.

## The right side = the left side on the reversed array

```
nums:   5 1 9 8 3 4        original index p  <->  rev index n-1-p
rev:    4 3 8 9 1 5
```

The right side runs right to left in the original, so left to right in `rev`. Build the same table on `rev`. For peak `p`, read `right[n-1-p][v]`.

## Combine at the peak

```
mountain:   [ left ... ]  PEAK  [ ... right ]
            |<-- L cell -->|
                           |<-- R cell -->|     the peak's cost is inside BOTH
```

```
answer = min over p in 1..n-2 and v in 1..V of
         L[p][v] + R[p][v] - |nums[p] - v|
```

Check on `[2,2,3]`, p = 1, v = 3: L = 1, R = 2, `|2-3| = 1`, so 1 + 2 - 1 = 2. ✓

## How high can v go?

Use `V = max(nums) + n`. This is safe: any part of the mountain above the max can be flattened to the smallest valid shape, whose peak is at most max + n/2 + 1. Too few columns can miss the answer (`[9,4,9]` needs 10 = max + 1). Too many columns only waste time.

Cost versus peak for `[3,3,3,3,3]`: peak 3 costs 6, peak 4 costs 3, peak 5 costs 4, peak 6 costs 9. It goes down, then up, with the minimum just above the max.

## Complexity

- **Time: O(n · V)**, with V = max + n. Two tables of n x V cells, each O(1) because of the running minimum, plus a final loop over n x V. At the limits it is about 2 million cells per table.
- **Space: O(n · V)** for the two tables, plus O(n) for the reversed array.

Without the running minimum, each cell would scan up to V cells, about 4 billion steps, which is too slow.

## Final code (tested)

Checked against all four examples and 600 random small arrays against a brute force, with no mismatches.

```java
class Solution {
    public int minOperations(int[] nums) {
        int n = nums.length;
        if (n < 3) {
            return -1;
        }
        int[] rev = new int[n];
        int INF = 1_000_000_000;
        for (int i = 0; i < n; i++) {
            rev[i] = nums[n - 1 - i];
        }

        int max = Integer.MIN_VALUE;
        for (int i : nums) {
            if (i > max) {
                max = i;
            }
        }
        int V = max + n;
        int[][] left = buildTable(nums, V, INF);
        int[][] right = buildTable(rev, V, INF);

        int ans = INF;
        for (int p = 1; p < n - 1; p++) {
            for (int v = 1; v <= V; v++) {
                int l = left[p][v];
                int r = right[n - 1 - p][v];
                if (l >= INF || r >= INF) continue;                  // impossible cell
                ans = Math.min(ans, l + r - Math.abs(nums[p] - v));  // peak counted twice, subtract once
            }
        }
        return ans;
    }

    private int[][] buildTable(int[] arr, int V, int INF) {
        int n = arr.length;
        int[][] table = new int[n][V + 1];
        for (int v = 1; v <= V; v++) {
            table[0][v] = Math.abs(arr[0] - v);
        }
        for (int i = 1; i < n - 1; i++) {                            // last row is never a peak, so never read
            int best = INF;
            for (int v = 1; v <= V; v++) {
                table[i][v] = (best >= INF) ? INF : Math.abs(arr[i] - v) + best;
                best = Math.min(table[i - 1][v], best);
            }
        }
        return table;
    }
}
```

## Mistakes and notes

| Item | What happened | Fix / rule |
|---|---|---|
| First idea | Raise the biggest middle element and patch a few others | Every element on both sides must be strictly monotonic. Choose the whole target array |
| Cost of a target | Unsure what to minimise | Sum of `|nums[i] - t[i]|` over the chosen target |
| INF vs 0 | Asked if 0 could mark impossible cells | 0 means free, the best cost. Use a big number, `1_000_000_000`. Avoid `Integer.MAX_VALUE` (overflow when adding) |
| Table width | Used `max + 1` columns | The peak can sit above the max: `V = max + n` |
| Row 0 | Wrote `nums[1]` for `v = 2` | Row 0 always uses `nums[0]`. Only `v` changes |
| Cell formula | Dropped `Math.abs` and `+ best` | `cell = |nums[i] - v| + best`, and stay `INF` if `best` is `INF` |
| best update | Compared against this row's change cost, then used `[v-1]` | `best = Math.min(best, table[i-1][v])`: previous row, same column, after computing the cell |
| Cheapest to the left | Thought it was the cell at `v-1` | It is the minimum of ALL cells to the left (running min) |
| Combine step | Read `left[p-1][v]` and `right[n-p][v]` | Read `left[p][v]` and `right[n-1-p][v]`. The cells already include the peak. Subtract the peak's cost once |
| Unfilled row | `right[n-p]` was an unfilled row of zeros | A zero cost looks free. Only read rows that are built (1 to n-2) |
| Syntax slips | Missing `;`, `Math.min[...]`, `INF` used before declared | Re-read the code line by line before pasting |
| My own slip | My first reason for the bound on `v` was wrong | The corrected reason is above: flatten any part above the max to the smallest valid shape |

## Worth knowing: four takeaways that carry over

### 1. When each choice depends on a neighbour's choice, store the best cost per (index, value)

You can't decide index 1 alone, because its choice limits index 0. Take `[5, 1]`, which must end strictly increasing:

```
If index 1 ends at 6 → index 0 may end anywhere below 6 → best is 5 (cost 0)
If index 1 ends at 2 → index 0 must end at 1          → cost 4
```

The cheapest way to handle index 1 depends on what you picked for index 0. So keep a table: one row per index, one column per final value, each cell the cheapest total so far. Same idea as "make an array non-decreasing with minimum cost."

### 2. Running minimum instead of rescanning

Every cell needs "the cheapest cell strictly to my left." Keep that number while walking across:

```
row above:     4    3    2    1    0
best so far:  INF   4    3    2    1    0
```

Rescanning for every column on a row of 2000 columns costs about 2000 x 2000 = 4 million steps. The running minimum costs 2000. Same idea as the stock problem: for each day you need the cheapest price **before** today, so you keep `minPrice` as you walk.

### 3. A mirrored side is the same problem on the reversed array

```
nums:  5 1 9 8 3 4
rev:   4 3 8 9 1 5       original index p  <->  rev index n-1-p
```

Seen again in Trapping Rain Water (tallest to my left and tallest to my right) and Product of Array Except Self (prefix pass and suffix pass). The second pass is the first pass run backwards.

### 4. Trace a tiny input by hand before coding

Three times a short trace caught a bug before the code ran:
- The first idea on `[1,1,1,1,1]` gives about 1, but the real answer is 4.
- The first Part 4 on `[2,2,3]` returned 0 instead of 2, because it read an unfilled row.
- Tracing row 1 of `[2,2,3]` showed `v = 2` costs 1, not 0, since `best` has to be added.

Use an input with an interior case (the peak, the middle, the neighbours behave differently). That is what exposes the slips.

## What I'd say in an interview

"I choose the cheapest target mountain. For each index and each final value I store the cheapest cost so far, using a running minimum for the cheapest strictly-smaller previous value. I do that from the left, and on the reversed array for the right, then combine at each peak and subtract the peak's own cost once. It's O(n times V) in time and space, where V is about the max plus n."
