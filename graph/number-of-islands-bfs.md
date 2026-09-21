# Number of Islands

**Pattern:** Graph Traversal — BFS on a Grid

## Problem
Given an `m x n` 2D grid of `'1'` (land) and `'0'` (water), count the number of
islands. An island is formed by land cells connected **horizontally or
vertically** (not diagonally).

LeetCode 200: https://leetcode.com/problems/number-of-islands/

## Approach — grid as an implicit graph
This is the same BFS mechanics as a standard adjacency-list graph (see
[[find-if-path-exists-in-graph-bfs]]), just with two things computed instead of
looked up:

- **Neighbors:** a cell `(row, col)` has exactly 4 potential neighbors —
  `(row-1,col)`, `(row+1,col)`, `(row,col-1)`, `(row,col+1)` — up/down/left/right
  only, no diagonals (per the problem's "horizontally or vertically" wording).
- **Bounds check:** before treating a candidate neighbor as valid, all four must
  hold: `newRow >= 0 && newRow < rows && newCol >= 0 && newCol < cols`. Off-by-one
  risk: valid indices for a dimension of size `rows` are `0` to `rows-1`, so the
  check is `newRow < rows`, not `<=`.
- **Visited tracking:** instead of a separate `Set`, mutate the grid directly —
  flip a visited land cell from `'1'` to `'0'`. This reuses the existing "is this
  land?" check (`grid[r][c] == '1'`) as the same check for "should I explore
  this?", and once flipped, it can never re-trigger a new BFS or be re-pushed.
  (Worth mentioning to an interviewer that this modifies the input in place — a
  common follow-up is "can you do it without modifying the input?", answered
  with a separate `boolean[][] visited` array instead.)
- **Mark visited at push time, not pop time** — same reasoning as the friendship/
  BFS trace from the previous problem: without this, the same cell could be
  pushed into the queue multiple times by different neighbors before either
  copy is popped.

## Algorithm
1. Outer double loop scans every cell `(i, j)`.
2. When a `'1'` is found: increment island count, flip it to `'0'`, push
   `[i, j]` into the queue, then run a BFS while-loop to consume the entire
   connected island.
3. Inside the BFS: pop `(row, col)`, check each of the 4 immediate neighbors —
   for each that passes bounds check AND is `'1'`, flip it to `'0'` and push it,
   together, in the same step.
4. Return the island count after the full grid scan completes.

**Self-correction during solving:** first attempt scanned as far as possible in
a straight line in each of the 4 directions per popped cell (more complex,
error-prone, and doesn't generalize). Simplified to the standard pattern: check
exactly the 4 immediate neighbors once per popped cell — same shape as a normal
adjacency-list BFS neighbor loop.

## Complexity
- **Time: O(m·n)** — the outer double loop is O(m·n) just to scan the grid.
  Across the *entire* program (all islands combined, not per-island), every
  cell can be flipped, pushed, and popped **at most once**, since a flipped
  `'0'` can never trigger a new BFS or be re-pushed again. So the total BFS work
  across all islands is also O(m·n), not O(m·n) *per* island. Combined:
  O(m·n) + O(m·n) = O(m·n).
- **Space: O(m·n)** — worst case, the `visited`/queue holds up to O(m·n) cells
  at once (e.g., the entire grid is one connected island). Note: if DFS with
  recursion were used instead of BFS with an explicit queue, the same O(m·n)
  bound would apply via the recursion call stack depth instead (e.g., a long,
  snake-like single-width island).

## Solution

```java
class Solution {
    public int numIslands(char[][] grid) {
        Queue<int[]> visited = new LinkedList<int[]>();
        int result = 0;

        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                if (grid[i][j] == '1') {
                    result++;
                    visited.add(new int[]{i, j});
                    grid[i][j] = '0';
                    while (!visited.isEmpty()) {
                        int[] arr = visited.remove();
                        int row = arr[0];
                        int col = arr[1];
                        if (row + 1 < grid.length && grid[row+1][col] == '1') {
                            visited.add(new int[]{row+1, col});
                            grid[row+1][col] = '0';
                        }
                        if (row - 1 >= 0 && grid[row-1][col] == '1') {
                            visited.add(new int[]{row-1, col});
                            grid[row-1][col] = '0';
                        }
                        if (col + 1 < grid[0].length && grid[row][col+1] == '1') {
                            visited.add(new int[]{row, col+1});
                            grid[row][col+1] = '0';
                        }
                        if (col - 1 >= 0 && grid[row][col-1] == '1') {
                            visited.add(new int[]{row, col-1});
                            grid[row][col-1] = '0';
                        }
                    }
                }
            }
        }
        return result;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Initially confused which direction was being varied — used variable names like `right`/`left`/`up`/`down` but the code actually varied row for "right" and column for "up" (mismatched naming vs. actual behavior) | When a variable name implies a direction, double check the code actually moves along the axis that direction implies (row = up/down, column = left/right) |
| First attempt scanned as far as possible in a straight line per direction (nested while loops per popped cell), rather than checking just the 4 immediate neighbors once | Overcomplicated the standard BFS pattern. The simpler, more general approach — check exactly the 4 immediate neighbors per popped cell — mirrors the adjacency-list BFS shape and is far less error-prone |
| Used `Array.of{...}` / `new int[2]` inside generic type brackets — invalid syntax | Correct array literal syntax is `new int[]{val1, val2}`; generic type declarations never take a size, just the type: `Queue<int[]>` |
| Briefly second-guessed whether "check all 4 directions and expand outward" meant this had to be DFS | Both BFS and DFS explore all 4 directions from every cell — the only difference is *how* you manage "what to explore next" (Queue = BFS, recursion/explicit Stack = DFS), not *what* gets explored |
