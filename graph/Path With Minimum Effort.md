# Path With Minimum Effort (Dijkstra on a grid)

**Problem:** LeetCode 1631. You're given a grid of heights. Start at the top-left cell and reach the bottom-right cell, moving up, down, left or right. The effort of a path is the **maximum absolute height difference between two consecutive cells** on it. Return the minimum effort over all paths.

**Link:** https://leetcode.com/problems/path-with-minimum-effort/

**Pattern:** Dijkstra on a grid. The cell is the node, the four moves are the edges, and the "cost" of a path is the **maximum** step, not the sum of the steps.

---

## Core insight

- The cost of a path is its biggest single step. Example grid:

```
1  2  2
3  8  2
5  3  5
```

```
Path A: 1 → 2 → 2 → 2 → 5     steps 1,0,0,3   effort 3
Path B: 1 → 3 → 5 → 3 → 5     steps 2,2,2,2   effort 2   <- better, even though the steps add up to more
```

- New effort for a neighbour = `max(effort so far, |height of popped cell - height of neighbour|)`. This is the one change from Network Delay Time, where it was "popped time + road time".
- Dijkstra still works because a path's effort **never goes down** when you extend it (one big step stays the worst step forever). It needs costs that never get better as you add a step, and max has that property just like sum.
- Scoreboard: an `efforts` grid of the same shape as the input, a **separate** array so the heights stay untouched. Start cell 0, every other cell `Integer.MAX_VALUE`.
- Heap entry: `{row, col, effort}`, ordered by effort.

## Plain-words plan

1. `efforts` grid: 0 at the start, `MAX_VALUE` elsewhere. Min-heap of `{row, col, effort}` with `{0, 0, 0}` in it.
2. While the heap isn't empty: pop. **Stale check:** if the popped effort is bigger than `efforts[row][col]`, skip.
3. For each of the four neighbours: bounds check first. Then `newEffort = max(effort so far, |height difference|)`. If `newEffort` is **strictly smaller** than the neighbour's stored value, update the grid **and** push `{r, c, newEffort}` (a new array).
4. When the heap is empty, return `efforts[m-1][n-1]`. (Optional: return as soon as the target cell is popped.)

Two separate comparisons, at different moments:
- the **stale check** right after the pop (popped effort vs the grid value for that cell)
- the **improve check** for each neighbour (new effort vs the neighbour's stored value)

## Hand trace on the example

| Pop `{r, c, effort}` | Neighbours updated | Notes |
|---|---|---|
| (0,0,0) | (1,0) = 2, (0,1) = 1 | |
| (0,1,1) | (1,1) = 6, (0,2) = 1 | |
| (0,2,1) | (1,2) = 1 | |
| (1,2,1) | (2,2) = 3 | (1,1): max(1,6) = 6, not smaller than 6, no update |
| (1,0,2) | (2,0) = 2, (1,1) = 5 | (1,1) improves from 6 to 5 |
| (2,0,2) | (2,1) = 2 | |
| (2,1,2) | (2,2) = 2 | (2,2) improves from 3 to 2 |
| (2,2,2) | none | the target: effort 2 |

The entries `(1,1,6)` and `(2,2,3)` are in the heap but stale when they pop, so they're skipped.

Final `efforts` grid:
```
0  1  1
2  5  1
2  2  2
```
Answer: 2 ✓.

## Complexity

C = m * n cells.

- **Time: O(C log C).** Map it from Network Delay Time (O(V + E log E)): V = C, and each cell has at most 4 neighbours, so E is at most 4C. Each edge causes at most one push, and each push or pop costs log of the heap size, which is at most log(4C), the same order as log C. The constant 4 drops out.
- **Space: O(C).** The `efforts` grid holds C numbers, and the heap holds at most 4C entries.

## Other ways to solve it (good for later)

- **Binary search on the answer + BFS:** for a limit `L`, BFS the grid using only steps with difference at most `L`. Binary search the smallest `L` that reaches the target. Same idea as Koko Eating Bananas. O(C log(max height)).
- **Union-Find:** sort all neighbour edges by height difference, merge cells in that order, and stop when the start and the target are connected. The difference of the last edge you add is the answer.

## Final code (tested)

Checked on the three LeetCode examples, a 1x1 grid, and 2000 random grids against a reference (binary search + BFS), with no mismatches.

```java
class Solution {
    public int minimumEffortPath(int[][] heights) {
        int m = heights.length;
        int n = heights[0].length;

        int rowArr[] = new int[]{1, 0, -1, 0};
        int colArr[] = new int[]{0, 1, 0, -1};

        int[][] efforts = new int[m][n];
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 && j == 0) continue;
                efforts[i][j] = Integer.MAX_VALUE;
            }
        }

        PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[2] - b[2]);   // {row, col, effort}
        minHeap.offer(new int[]{0, 0, 0});

        while (minHeap.size() > 0) {
            int[] a = minHeap.poll();
            int row = a[0];
            int col = a[1];
            int oldEffort = efforts[row][col];

            if (a[2] > oldEffort) continue;                           // stale entry
            int height = heights[row][col];
            for (int i = 0; i < 4; i++) {
                int neighbourRow = row + rowArr[i];
                int neighbourCol = col + colArr[i];
                if (neighbourRow >= 0 && neighbourRow < m && neighbourCol >= 0 && neighbourCol < n) {
                    int neighbourHeight = heights[neighbourRow][neighbourCol];
                    int newNeighbourEffort = Math.max(oldEffort, Math.abs(height - neighbourHeight));
                    if (newNeighbourEffort < efforts[neighbourRow][neighbourCol]) {
                        efforts[neighbourRow][neighbourCol] = newNeighbourEffort;
                        minHeap.offer(new int[]{neighbourRow, neighbourCol, newNeighbourEffort});   // push a NEW array
                    }
                }
            }
        }
        return efforts[m - 1][n - 1];
    }
}
```

## Mistakes and notes

| Item | What happened | Fix / rule |
|---|---|---|
| Jargon | "Scoreboard indexed by" wasn't clear | The scoreboard has one slot per node. On a grid a node is a cell, so the scoreboard is a grid of the same shape |
| New cost | Described it as "popped score plus something" | Here it is `max(effort so far, |height difference|)`, because a path's cost is its biggest step |
| Two comparisons mixed | Said "if the height difference is greater, no need to push" | First combine with the max. Then compare the result with the neighbour's stored value. Push only if strictly smaller |
| Update direction | Asked if you update when the new effort is greater | Update only when the new effort is **smaller**. The stored value is the best (lowest) so far |
| Name clash | Arrays `row[]` and `col[]` clashed with `int row`, `int col` | Name the direction arrays `rowArr`/`colArr` (or `dr`/`dc`) |
| Typo | `PrioirtyQueue` | Re-read the class names before pasting |
| Self-reference | `int height = height[row][col];` | Read from `heights[row][col]` |
| Stale check | Read both values from the grid, so it never fired. Then flipped it | Stale means the **popped** effort `a[2]` is bigger than the grid's value for the cell |
| Push outside the `if` | Pushed every in-bounds neighbour every time, which loops forever on a grid | Update and push **together**, inside the improve check |
| Deleted variable | Removed `effort` but still used it | After deleting a line, search for the name |
| Missing return | No return after the loop | `return efforts[m - 1][n - 1];` |
| Complexity | Couldn't derive it directly | Map to the problem you know: V = C cells, E at most 4C, time O(V + E log E) becomes O(C log C), space O(C) |
