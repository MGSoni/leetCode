# Max Area of Island

**Pattern:** Graph Traversal — DFS (Recursive)

## Problem
Given a grid of `1` (land) and `0` (water), 4-directionally connected, return
the area (cell count) of the **largest** connected island. Return `0` if there
is no land at all.

LeetCode 695: https://leetcode.com/problems/max-area-of-island/

## DFS vs BFS — core distinction
BFS explores level by level (all direct neighbors before going deeper), needing
an explicit `Queue` for FIFO order. DFS dives as deep as possible down **one**
path before backtracking — recursion's **call stack** naturally provides this
backtracking behavior for free, with no explicit `Stack` needed.

## Approach — recursive area accumulation
Think of it recursively: the area of the island starting at a cell equals 1
(for the cell itself) plus the areas contributed by each of its 4 neighbors,
computed the same way, recursively.

```
area(r, c):
    if out of bounds OR grid[r][c] != 1: return 0
    grid[r][c] = 0                          // mark visited BEFORE recursing
    return 1 + area(r-1,c) + area(r+1,c) + area(r,c-1) + area(r,c+1)
```

**Why marking must happen before recursing, not after:** traced a minimal
2-cell example (`(0,0)` and `(0,1)` both land) without any visited-marking —
`area(0,0)` calls `area(0,1)`, which calls back `area(0,0)`, which calls
`area(0,1)` again, forever. Marking the current cell `0` *before* making any
recursive calls to neighbors ensures that by the time a neighbor's call looks
back at this cell, it's already excluded.

**Key simplification:** the base case (bounds check + "is this land?" check)
should live **once, at the very top** of the function, checking the cell the
function was *called with* — not repeated inside the loop checking something
else. Once that's true, the loop over 4 neighbors can just call
`dfs(grid, r, c, ...)` unconditionally for each neighbor and let the
**recursive call's own base case** handle both bounds-checking and the
land-check — no need for manual pre-checks inside the loop at all.

## Algorithm
1. Outer double loop scans every cell. When a `1` is found, call `dfs(i, j)`
   and compare its return value against a local `maxArea` variable
   (`Math.max(maxArea, area)`) — no class-level/global field needed, since
   `dfs` returns a value directly.
2. `dfs(row, col)`: bounds/land check → mark visited → sum `1 +` the 4
   recursive neighbor calls → return the total.

## Complexity
- **Time: O(m·n)** — same reasoning as Number of Islands: once a cell is
  marked `0`, it can never be processed again by any future call (own island's
  recursion or a later outer-scan trigger), so total DFS work across the
  entire grid is O(m·n), combined with the O(m·n) outer scan.
- **Space: O(m·n)** — no separate visited structure is used (the grid is
  mutated directly, contributing O(1) auxiliary space), but the **recursion
  call stack** can grow up to O(m·n) deep in the worst case: a single,
  snake-like island winding through nearly the entire grid, where each cell's
  `dfs` call nests one level deeper before any of them return. Worth noting:
  this is the same overall bound as the BFS version of Number of Islands
  (also O(m·n)), but the *mechanism* differs — BFS's cost comes from an
  explicit `Queue`; DFS's cost comes from the *implicit* call stack.

## Solution

```java
class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int maxArea = 0;
        int[] rowArr = new int[]{1,0,-1,0};
        int[] colArr = new int[]{0,1,0,-1};
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                if (grid[i][j] == 1) {
                    int area = dfs(grid, i, j, rowArr, colArr);
                    maxArea = Math.max(area, maxArea);
                }
            }
        }
        return maxArea;
    }

    public int dfs(int[][] grid, int row, int col, int[] rowArr, int[] colArr) {
        if (row < 0 || row >= grid.length || col < 0 || col >= grid[0].length) {
            return 0;
        }
        if (grid[row][col] == 0) return 0;

        int area = 0;
        grid[row][col] = 0;
        for (int i = 0; i < 4; i++) {
            area += dfs(grid, row + rowArr[i], col + colArr[i], rowArr, colArr);
        }
        return area + 1;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Declared `dfs` as `void` but had `return` statements with values inside it | Method return type must match what's actually being returned — needed `int`, since the caller uses the returned area count |
| First base case only checked bounds, missing the "is this land?" (`grid[row][col] != 1`) check entirely | The base case needs to handle both "out of bounds" AND "not land / already visited" in one place — visited cells are flipped to `0`, so this single check covers both concerns |
| Missing the "mark visited" step (`grid[row][col] = 0`) entirely in an early draft | Without it, the recursion has no way to stop calling back and forth between adjacent cells — traced a concrete 2-cell infinite-recursion example to confirm |
| Used `area = 1 + dfs(...)` (assignment) inside a loop over 4 neighbors instead of `area += dfs(...)` | Assignment overwrites the previous iteration's result instead of accumulating; needed `+=` to sum all 4 neighbor contributions |
| Put the "is this land?" check and the "mark visited" step *inside* the loop, checking the *current* cell (`grid[row][col]`) repeatedly instead of checking each *neighbor* before recursing into it | This caused checking/marking the wrong cell on each iteration — after marking the current cell `0` on the first iteration, subsequent iterations' `if` check failed and silently skipped recursing into other valid neighbors |
| Manually bounds-checked and land-checked each neighbor inside the loop before calling `dfs` on it | Redundant — the recursive call's own base case already handles both checks. Simplified by calling `dfs(grid, r, c, ...)` unconditionally for all 4 neighbors and trusting the base case |
