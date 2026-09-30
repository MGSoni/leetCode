# Number of Closed Islands

**Pattern:** Graph Traversal — DFS, Two-Pass Technique

## Problem
Grid where `0` = land, `1` = water (note: flipped convention from typical
island problems). An island is a connected group of `0`s. A **closed** island
never touches the grid's border. Count the number of closed islands.

LeetCode 1254: https://leetcode.com/problems/number-of-closed-islands/

## Core insight — the two-pass technique
The tricky part: "does this island touch the border" isn't known at the
*start* of exploring an island — it might only become true partway through the
DFS, or never. Trying to detect and propagate "touched the border" information
back up through recursive calls mid-traversal gets messy.

**Cleaner solution: do it in two separate passes.**
1. **Pass 1 — eliminate border-touching land first.** Before counting anything,
   run DFS starting from every land cell sitting directly *on* the border
   (row 0, last row, col 0, last col). Flip all of those cells — and
   everything connected to them — to water (`1`). This guarantees that after
   Pass 1, **any remaining land cell cannot possibly be connected to the
   border** (if it were, it would already have been flooded away).
2. **Pass 2 — now it's just standard Number of Islands.** Scan the grid
   normally; every remaining `0` found is, by construction, a **closed**
   island. Flood-fill and count each one the same way as the standard problem.

This sidesteps ever having to ask "was this a closed island?" mid-recursion —
border-touching islands are simply removed *before* counting begins.

## DFS helper — same simplification as Max Area of Island
```
dfs(row, col):
    if out of bounds: return
    if grid[row][col] == 1: return    // water or already-visited
    grid[row][col] = 1                // mark visited BEFORE recursing
    for each of 4 neighbors (newRow, newCol):
        dfs(newRow, newCol)           // unconditional — let the base case
                                        // handle bounds + water/visited check
```
No manual bounds-check or land-check needed inside the loop — the recursive
call's own base case handles both, same pattern established in
[[max-area-of-island-dfs]].

## Complexity
- **Time: O(m·n)** — once a cell is flipped to `1` (water), it can never be
  visited again by *any* later DFS call, whether from Pass 1's remaining
  border cells or Pass 2's scan. So total DFS work across *both* passes
  combined is still bounded by O(m·n) — each cell touched at most once, ever,
  regardless of how many separate passes or starting points trigger DFS calls.
- **Space: O(m·n)** — same worst-case reasoning as Max Area of Island: a
  single large connected region (up to the whole grid) could produce a
  recursion call stack depth of O(m·n) in the worst case.

## Solution

```java
class Solution {
    public int closedIsland(int[][] grid) {
        int[] r = new int[]{1,0,-1,0};
        int[] c = new int[]{0,1,0,-1};

        // Pass 1: flood-fill away any island touching the border
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[0][j] == 0) dfs(grid, 0, j, r, c);
            if (grid[grid.length-1][j] == 0) dfs(grid, grid.length-1, j, r, c);
        }
        for (int i = 0; i < grid.length; i++) {
            if (grid[i][0] == 0) dfs(grid, i, 0, r, c);
            if (grid[i][grid[0].length-1] == 0) dfs(grid, i, grid[0].length-1, r, c);
        }

        // Pass 2: count remaining (closed) islands
        int count = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                if (grid[i][j] == 0) {
                    count++;
                    dfs(grid, i, j, r, c);
                }
            }
        }
        return count;
    }

    private void dfs(int[][] grid, int row, int col, int[] r, int[] c) {
        if (row < 0 || row >= grid.length || col < 0 || col >= grid[0].length) {
            return;
        }
        if (grid[row][col] == 1) return;
        grid[row][col] = 1;
        for (int i = 0; i < 4; i++) {
            int newRow = row + r[i];
            int newCol = col + c[i];
            dfs(grid, newRow, newCol, r, c);
        }
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Inside the neighbor loop, checked `if(grid[row][col]==0)` (the *current* cell, already just marked `1`) instead of checking the *neighbor* before recursing, and called `dfs(grid, row, col, ...)` (current cell again) instead of `dfs(grid, newRow, newCol, ...)` | Same mistake pattern as an earlier DFS problem (Max Area of Island) — checking/passing the wrong variable inside a loop. Fixed by dropping the manual check entirely and calling `dfs` unconditionally on the computed neighbor coordinates, trusting the recursive call's own base case to handle bounds + already-visited |

**Process note:** this problem was solved with pseudocode written and discussed
*before* any code, and only one bug appeared in the first full attempt (fixed
immediately on the next try) — a marked improvement over earlier problems this
session that took several rounds of debugging. Confirms that writing the plan
out first, then hand-tracing before pasting code, meaningfully reduces bug
count.
