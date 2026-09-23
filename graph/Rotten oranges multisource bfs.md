# Rotten Oranges

**Pattern:** Graph Traversal — Multi-Source BFS with Level/Batch Tracking

## Problem
Given an `N x M` grid where `0` = empty, `1` = fresh orange, `2` = rotten orange.
Every minute, any fresh orange adjacent (4-directionally) to a rotten orange
becomes rotten. Return the minimum minutes until no fresh orange remains, or
`-1` if impossible.

## Key ideas beyond standard BFS

**1. Multi-source BFS.** Unlike previous problems with a single `source`, there
can be multiple rotten oranges at minute 0. All of them rot their neighbors
*simultaneously*, one layer at a time — so seed the BFS queue with **every**
rotten cell found in the initial scan, not just one.

**2. Level/batch tracking to count minutes.** A naive "increment minutes every
push" badly overcounts (one popped node can push several neighbors, which
shouldn't count as several minutes). The correct pattern:
```java
while (!queue.isEmpty()) {
    int batchSize = queue.size();     // snapshot BEFORE popping anything
    for (int i = 0; i < batchSize; i++) {
        // pop exactly one node, process it, push new neighbors
        // (new pushes land at the back of the queue — don't affect batchSize,
        //  since batchSize was already captured and the for-loop uses its own
        //  counter, not queue.size() again)
    }
    // exactly `batchSize` nodes popped = one full level/minute processed
}
```

**3. Only increment `minutes` if the batch actually rotted something new.**
Tested against the edge case `A = [[2]]` (single rotten orange, no fresh
oranges): incrementing unconditionally after every batch gives `minutes = 1`
(wrong — the correct answer is `0`, since nothing needed to rot). The fix: a
`boolean hasRottenAnotherOrange` flag, reset to `false` at the **start of each
batch** (must be declared *outside* the inner `for` loop but *inside* the outer
`while` loop, so it persists across the whole batch and resets each round), set
`true` whenever any neighbor is rotted during that batch, and checked after the
batch to decide whether to increment `minutes`.
Verified this doesn't break with multiple rotten clusters of different sizes:
as long as any cluster is still actively spreading, the queue still contains
items and that batch still rots something new — the flag only correctly
suppresses the increment when truly nothing happened.

**4. Track "is it impossible" without a second scan.** Count `freshOranges`
once during the initial scan, decrement it every time a fresh orange is
rotted during BFS. At the end, if `freshOranges > 0`, some oranges were
unreachable → return `-1`.

**5. Offset arrays for 4-directional neighbors.** Instead of 4 separate `if`
blocks (as done in Number of Islands), use:
```java
int row[] = {-1, 0, 1, 0};
int col[] = {0, -1, 0, 1};
```
and loop `j` from `0` to `3`, computing `rowIndex = arr[0]+row[j]`,
`colIndex = arr[1]+col[j]` — a cleaner, more scalable pattern for checking all
4 neighbors.

## Complexity
- **Time: O(N·M)** — initial scan is O(N·M). BFS traversal: every cell can be
  enqueued/dequeued at most once (once rotted, a cell can never rot again or
  re-enter the queue), so total BFS work is also O(N·M). Combined:
  O(N·M) + O(N·M) = O(N·M).
- **Space: O(N·M)** — worst case, the queue holds close to every cell at some
  point (e.g., grid alternates such that rot spreads to nearly everything).
  The `row[]`/`col[]` offset arrays are fixed size 4 → O(1), doesn't change
  the complexity class.

## Solution

```java
public class Solution {
    public int solve(int[][] A) {
        int freshOranges = 0;
        Queue<int[]> rottenOrangeCell = new LinkedList<>();
        int row[] = new int[]{-1,0,1,0};
        int col[] = new int[]{0,-1,0,1};

        for (int i = 0; i < A.length; i++) {
            for (int j = 0; j < A[0].length; j++) {
                int value = A[i][j];
                if (value == 2) {
                    rottenOrangeCell.add(new int[]{i,j});
                } else if (value == 1) {
                    freshOranges++;
                }
            }
        }

        int minutes = 0;
        while (!rottenOrangeCell.isEmpty()) {
            int size = rottenOrangeCell.size();
            boolean hasRottenAnotherOrange = false;
            for (int i = 0; i < size; i++) {
                int arr[] = rottenOrangeCell.remove();
                for (int j = 0; j < 4; j++) {
                    int rowIndex = arr[0] + row[j];
                    int colIndex = arr[1] + col[j];
                    if (rowIndex >= 0 && rowIndex < A.length && colIndex >= 0 && colIndex < A[0].length &&
                        A[rowIndex][colIndex] == 1) {
                        freshOranges--;
                        A[rowIndex][colIndex] = 2;
                        hasRottenAnotherOrange = true;
                        rottenOrangeCell.add(new int[]{rowIndex, colIndex});
                    }
                }
            }
            if (hasRottenAnotherOrange)
                minutes++;
        }

        if (freshOranges > 0) {
            return -1;
        }
        return minutes;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Initially planned to increment `minutes` on every push into the queue | Overcounts badly — one popped node pushing 3 neighbors isn't 3 minutes. Fixed by processing in batches (snapshot `queue.size()` before popping) and incrementing once per batch instead |
| First full attempt incremented `minutes` unconditionally after every batch | Traced against `A = [[2]]` (single rotten orange, no fresh oranges) — unconditional increment gives `minutes = 1`, but correct answer is `0`. Fixed with a `hasRottenAnotherOrange` flag, only incrementing if the batch actually rotted something |
| Declared `hasRottenAnotherOrange` *inside* the inner `for` loop, then tried to read it *outside* that loop (in the `if` check after) | A variable's scope ends with its enclosing block. Needed to declare it once per **batch** — outside the inner `for`, inside the outer `while` — so it persists for the whole batch and resets correctly each round |
| Bounds check used mismatched comparison operators: `rowIndex<A.length` but `colIndex<=A[0].length` | For a dimension of size `A[0].length`, valid indices are `0` to `A[0].length-1`, so the check must be `colIndex<A[0].length` (strict `<`, not `<=`) — same off-by-one risk as any bounds check, worth double-checking both dimensions use the same operator |
| Missing bounds check entirely on first attempt, before checking `A[rowIndex][colIndex]==1` | Always bounds-check candidate neighbor coordinates *before* accessing the array with them, or risk `ArrayIndexOutOfBoundsException` |
