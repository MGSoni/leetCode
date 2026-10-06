# Pacific Atlantic Water Flow

**Pattern:** Multi-source BFS on a grid, searching in reverse from the borders, then intersecting two visited sets

## Problem
Given an `m x n` grid of heights, the Pacific Ocean touches the top row and left
column, and the Atlantic Ocean touches the bottom row and right column. Water
flows from a cell to a 4-directional neighbor only if the neighbor's height is
**equal or lower**. Return every cell from which water can reach **both** oceans.

LeetCode 417: https://leetcode.com/problems/pacific-atlantic-water-flow/

## Core insight: search from the oceans inward, not from every cell outward
- Brute force starts a search from every cell and checks both oceans. Each search
  can touch the whole grid, so that is about O((m·n)^2).
- Flip the direction. Start from the ocean's border cells and walk **uphill**
  (move to a neighbor only if its height is `>=` the current cell). Every cell
  such a search reaches can drain back down to that ocean.
- A search from the border only tells you about **that** ocean, so run **two
  separate searches**: one from the Pacific border, one from the Atlantic border.
- The answer is the **intersection**: cells visited by both searches.
- Starting at a border cell and flowing downhill does not work. It answers where
  that border cell's own water goes, and the interior cells are never the start.

## Why the grid can't be the visited marker here
Earlier grid problems flipped `1` to `0` in the grid. That fails here for two reasons:
1. There are **two** searches, and at the end you need both visited records at
   the same time. Restoring the grid between searches loses the first record.
2. The heights are real data that the neighbor rule compares (heights go up to
   10^5 on this problem), so overwriting them to mark visited breaks the
   comparison.

Use two separate `boolean[m][n]` matrices, one per ocean. Heights stay untouched.

## Algorithm
1. Create `pacificVisited` and `atlanticVisited`, each `boolean[m][n]`, and one
   queue per ocean.
2. Seed the Pacific queue with row `0` and column `0`. Seed the Atlantic queue
   with row `m-1` and column `n-1`. (The last row index is `m-1` and the last
   column index is `n-1`. Index `n` doesn't exist, and the grid isn't necessarily
   square.)
3. Seed with check-then-mark: `if (!visited[r][c]) { visited[r][c] = true; queue.add(...) }`.
   Corner cells are hit by both loops of the same ocean, and the second time they
   are skipped. The top-right and bottom-left corners belong to both oceans, so
   they end up marked in both matrices.
4. Run one BFS per ocean. For each popped cell, loop over the 4 offsets and push
   a neighbor if: it is in bounds, it is not yet visited by this search, and
   `heights[neighbor] >= heights[current]`. Mark at push time.
5. After both BFS runs, add every cell that is `true` in both matrices to the result.

**BFS is a good fit** here. TC is the same either way, but BFS uses an explicit
queue instead of recursion, so it avoids deep recursion on a large grid.

## Hand trace, 3x3 grid with a high center
```
[[1, 1, 1],
 [1, 3, 1],
 [1, 1, 1]]
```
- Pacific seeds: top row and left column. From `(0,1)`, the center `(1,1)` has
  `3 >= 1`, so it is visited.
- Atlantic seeds: bottom row and right column. From `(2,1)`, the center is also
  visited the same way.
- Every other cell is height 1 and reachable from a border cell of both oceans, so
  both searches visit all 9 cells and the answer is all 9 cells.

Use this kind of grid when checking. On a 2x2 grid every cell is a border cell, so
the BFS has nothing to discover and a broken BFS still looks right.

## Complexity
- **Time: O(m·n)**. Each cell is pushed at most once per search, since it is marked
  at push time, and each pop does 4 constant-time checks. Two BFS runs plus the
  final scan stay O(m·n).
- **Space: O(m·n)**. Two boolean matrices of m·n each, plus queues that can hold up
  to m·n cells. The result list can also reach m·n entries (output).

## Solution

```java
class Solution {
    public List<List<Integer>> pacificAtlantic(int[][] heights) {

        Queue<int[]> pacificQueue = new LinkedList<>();
        Queue<int[]> atlanticQueue = new LinkedList<>();

        boolean[][] pacificVisited = new boolean[heights.length][heights[0].length];
        boolean[][] atlanticVisited = new boolean[heights.length][heights[0].length];

        for (int j = 0; j < heights[0].length; j++) {
            int[] p = new int[]{0, j};
            int[] a = new int[]{heights.length - 1, j};
            if (!pacificVisited[0][j]) {
                pacificQueue.add(p);
                pacificVisited[0][j] = true;
            }
            if (!atlanticVisited[heights.length - 1][j]) {
                atlanticQueue.add(a);
                atlanticVisited[heights.length - 1][j] = true;
            }
        }

        for (int i = 0; i < heights.length; i++) {
            int[] p = new int[]{i, 0};
            int[] a = new int[]{i, heights[0].length - 1};
            if (!pacificVisited[i][0]) {
                pacificQueue.add(p);
                pacificVisited[i][0] = true;
            }
            if (!atlanticVisited[i][heights[0].length - 1]) {
                atlanticQueue.add(a);
                atlanticVisited[i][heights[0].length - 1] = true;
            }
        }

        bfs(heights, pacificVisited, pacificQueue);
        bfs(heights, atlanticVisited, atlanticQueue);

        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < heights.length; i++) {
            for (int j = 0; j < heights[0].length; j++) {
                if (pacificVisited[i][j] && atlanticVisited[i][j]) {
                    result.add(List.of(i, j));
                }
            }
        }
        return result;
    }

    private void bfs(int[][] heights, boolean[][] visited, Queue<int[]> queue) {
        int[] rowArr = new int[]{1, 0, -1, 0};
        int[] colArr = new int[]{0, 1, 0, -1};

        while (!queue.isEmpty()) {
            int[] arr = queue.remove();
            int row = arr[0];
            int col = arr[1];
            for (int i = 0; i < 4; i++) {
                int newRow = row + rowArr[i];
                int newCol = col + colArr[i];
                if (newRow >= 0 && newRow < heights.length && newCol >= 0 && newCol < heights[0].length
                        && heights[newRow][newCol] >= heights[row][col]) {
                    if (!visited[newRow][newCol]) {
                        queue.add(new int[]{newRow, newCol});
                        visited[newRow][newCol] = true;
                    }
                }
            }
        }
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| First idea: start at a border cell and flow downhill, or search from one ocean's border to reach the other | Search in reverse (uphill) from each ocean's border. Run two separate searches, and the answer is the intersection of their visited sets |
| Proposed marking visited by subtracting a constant from the heights | Fails: heights go up to 10^5, the neighbor rule needs the real heights, and the second search overwrites the first one's record. Use two boolean matrices |
| Used index `n` for the last row and column of the Atlantic seed | Last indices are `m-1` (rows) and `n-1` (columns). Keep rows and columns separate, since the grid isn't necessarily square |
| Seed code had compile errors: `j<heights[0]`, `new int{...}`, `height` vs `heights`, a misspelled queue name | Loop bounds need `.length`, array literals need `new int[]{...}`, and variable names must match their declarations exactly |
| `if(visited[row][col]) continue;` right after popping a cell | Cells are marked visited when pushed, so every popped cell is already marked. The `continue` fired on every pop and the BFS never expanded past the border. Marking at push time already prevents duplicates, so the pop-time check isn't needed |
| The 2x2 hand trace would not have exposed that bug | On a 2x2 grid every cell is a border cell. Trace on a grid with an interior cell, such as the 3x3 grid above |
