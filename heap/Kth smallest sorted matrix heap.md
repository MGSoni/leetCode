# Kth Smallest Element in a Sorted Matrix

**Pattern:** K-way merge with a min-heap (each row is a sorted list)

## Problem
An `n x n` matrix has every row and every column sorted ascending. Return the k-th smallest
element in the whole matrix. It is the k-th in sorted order, so duplicates count separately.

Example: `matrix = [[1,5,9],[10,11,13],[12,13,15]]`, `k = 8` -> `13`

LeetCode 378: https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/
Constraint: `n <= 300`, `1 <= k <= n^2`

## Approaches
| Approach | Time | Extra space | Notes |
|---|---|---|---|
| Flatten, sort, take index `k - 1` | O(n^2 log n) | O(n^2) | Simplest. Ignores the sorted structure |
| **Min-heap over the rows (K-way merge)** | O((n + k) log n) | O(n) | Used here |
| Binary search on the value | O(n log(max - min)) | O(1) | Follow-up, not practiced yet |

Notes on the first row: insertion sort by hand is O(N^2), and a library sort is O(N log N), where
N = n^2 is the number of elements. The k-th smallest is at index `k - 1` in a 0-based array.

## Core insight
**Treat each row as its own sorted list.** The example matrix is three sorted lists:
`1, 5, 9` and `10, 11, 13` and `12, 13, 15`. The k-th smallest in the matrix is the k-th smallest
you get by **merging those sorted lists**. Only the front of each list can be the next smallest,
because everything behind it in the same row is larger. This is the same pattern as Merge K Sorted
Lists.

- **Sorted rows** tell you what to push next: the next element in the same row.
- **The comparator** decides which of the entries currently in the heap comes out next. Those
  entries come from different rows, and nothing else says how a `1` from row 0 compares with a
  `10` from row 1. The heap has to compare them.

## The heap entry
Each entry is an `int[]` with three numbers: `{value, row, col}`.
- `entry[0]` = value, what the heap orders by
- `entry[1]` = row, which row it came from
- `entry[2]` = col, where it is in that row

After a pop, `row` and `col` tell you the next element of that row, `matrix[row][col + 1]`. Without
them a popped `5` is just a number.

```java
PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));
```
`Integer.compare` is safer than `a[0] - b[0]`, because subtracting two values near a billion in
magnitude can overflow. Without a comparator, a `PriorityQueue<int[]>` throws as soon as it has to
compare two arrays.

## Algorithm
```
for each row i: heap.offer({matrix[i][0], i, 0})     // one entry per row, at column 0
repeat k times:
    cur = heap.poll()                                // next smallest in the whole matrix
    if this was the k-th pop: return cur[0]
    if cur.col + 1 < number of columns:
        heap.offer({matrix[cur.row][cur.col + 1], cur.row, cur.col + 1})
```
After every pop, push the next element from the same row, if the row has one left. The answer is
the value popped on the k-th pop. Stop there, with no more pushes.

## Hand trace, `[[1,5,9],[10,11,13],[12,13,15]]`, `k = 4`
| Step | Pop | Push | Heap values after |
|---|---|---|---|
| start | | | 1, 10, 12 |
| pop 1 | 1 | 5 | 5, 10, 12 |
| pop 2 | 5 | 9 | 9, 10, 12 |
| pop 3 | 9 | none (row 0 used up) | 10, 12 |
| pop 4 | 10 | not needed, return it | |

The answer is `10`. With `k = 3` the answer is `9`, the third pop.

## Complexity
- **Time: O((m + k) log m)**, written `O((n + k) log n)` for an `n x n` matrix, where `m` is the
  number of rows. The initial fill does `m` offers at O(log m) each, which is O(m log m). The loop
  does at most `k` polls and `k` offers, each O(log m), which is O(k log m). A heap operation
  costs the log of the heap's size, not its size. When `k >= m` the loop dominates, which is why
  `O(k log m)` is a reasonable shorthand, but with a small `k` the initial fill is the larger term.
- **Space: O(m)**, which is O(n). The heap holds one entry per row at the start, and each pop is
  followed by at most one push, so it never grows. It does not depend on `k`.
- **Versus sorting:** the heap is much faster when `k` is small. When `k` is near `n^2`, the time
  is about the same as sorting, O(n^2 log n), but the heap still wins on space: O(n) against
  O(n^2).

## Solution

```java
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));

        for (int i = 0; i < matrix.length; i++) {
            minHeap.offer(new int[]{matrix[i][0], i, 0});
        }
        int count = 0;
        while (count < k) {
            int[] min = minHeap.poll();
            int row = min[1];
            int col = min[2];
            count++;
            if (count == k) {
                return min[0];
            }
            if (col + 1 < matrix[0].length) {
                minHeap.offer(new int[]{matrix[row][col + 1], row, col + 1});
            }
        }
        return -1;
    }
}
```
The `return -1` is unreachable, because `k <= n^2` guarantees the loop returns. Java needs it anyway.

## Mistakes and confusions while working through it

| What went wrong | Fix / insight |
|---|---|
| Called insertion sort O(n log n) | Hand insertion sort is O(N^2). A library sort is O(N log N). Say which one |
| Believed sorted rows meant no comparator was needed | Sorted rows tell you what to push next. The comparator picks between entries from different rows |
| First trace popped `1` and never pushed `5` | After every pop, push the next element from the same row |
| Did not know what a heap entry should carry | `{value, row, col}`. The row and column are what let you find the next element |
| Gave TC as O(k) and SC as O(k) | The heap holds one entry per row, so SC is O(m), not O(k). Each operation costs log of the heap size |
| Thought the cost was O(m * k) | A heap operation costs O(log m), not O(m) |
| Gave TC as O(k log m), missing the initial fill | The full cost is O((m + k) log m), because `m` offers happen before the loop |
