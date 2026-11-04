# Kth Largest Element in an Array

**Pattern:** Heap, top-K (min-heap of size k)

## Problem
Given an integer array `nums` and an integer `k`, return the k-th largest element in the array.
It is the k-th largest in sorted order, so duplicates count separately.

Example: `nums = [3,2,1,5,6,4]`, `k = 2` -> `5`
Example: `nums = [3,2,3,1,2,4,5,5,6]`, `k = 4` -> `4`

LeetCode 215: https://leetcode.com/problems/kth-largest-element-in-an-array/
Constraint: `1 <= k <= nums.length <= 100,000`

## Approaches
| Approach | Time | Extra space | Notes |
|---|---|---|---|
| Sort, then take index `n - k` | O(n log n) | O(log n) for `int[]` (dual-pivot quicksort stack) | Simplest. Passes at this constraint |
| Min-heap of size `k` | O(n log k) | O(k) | Best when `k` is much smaller than `n` |
| Quickselect | O(n) average, O(n^2) worst | O(1) to O(log n) | Not practiced yet |

- **Index:** with a 0-based array sorted ascending, the answer is at index `n - k`. Check:
  `[1,2,3,4,5,6]` with `k = 2` gives `nums[4] = 5`. `n - k + 1` is the 1-based position.
- **Space of sorting:** O(log n) for `int[]`. It would be O(n) for `Integer[]` or a `List` (TimSort).

## Core insight
The k-th largest element is **the smallest of the k largest elements**. So keep only the k
largest numbers seen so far in a **min-heap**. Its top is the smallest of those k, which is the
answer.

**A min-heap is not fully sorted.** Only the top is guaranteed to be the smallest, and the rest
sit in no particular order. You never read the heap in order, you only look at the top.

**Why a min-heap and not a max-heap:** when a new number arrives you must decide whether it beats
the **smallest** number you're holding. A min-heap gives you that number in O(1) and removes it in
O(log k). A max-heap would put the largest number on top, which you never need.

## Algorithm
```
for each number x in nums:
    if heap.size() < k:        // still filling up
        heap.offer(x)
    else if x > heap.peek():   // heap is full and x beats the smallest held number
        heap.poll()
        heap.offer(x)
    // else: x is not among the k largest, ignore it
return heap.peek()
```
The "ignore if smaller than the top" rule starts only once the heap holds `k` numbers. Before
that, every number goes in, even a small one.

## Hand traces
**`[3,2,1,5,6,4]`, `k = 2`:** heap after each element is `{3}`, `{2,3}`, `{2,3}` (1 rejected),
`{3,5}`, `{5,6}`, `{5,6}` (4 rejected). Top is `5`.

**`[3,2,3,1,2,4,5,5,6]`, `k = 4`:**

| Arrives | Action | Heap after (top first) |
|---|---|---|
| 3 | fewer than 4, add | 3 |
| 2 | fewer than 4, add | 2, 3 |
| 3 | fewer than 4, add | 2, 3, 3 |
| 1 | fewer than 4, add | 1, 2, 3, 3 |
| 2 | beats top 1, replace | 2, 2, 3, 3 |
| 4 | beats top 2, replace | 2, 3, 3, 4 |
| 5 | beats top 2, replace | 3, 3, 4, 5 |
| 5 | beats top 3, replace | 3, 4, 5, 5 |
| 6 | beats top 3, replace | 4, 5, 5, 6 |

Top is `4`, which matches the expected answer.

## Complexity
- **Time: O(n log k).** Each element does at most one `peek` (O(1)), one `poll` and one `offer`.
  The heap never holds more than `k` numbers, so `poll` and `offer` each cost O(log k).
- **Space: O(k).** The heap holds at most `k` numbers. The `else` branch only runs while the size is
  below `k`, and the replace branch does a `poll` and an `offer`, so the size stays at `k`.
  For `k = 2` this is O(1), not O(n).
- **Heap vs sort:** with `k = 2` and `n = 100,000` the heap does about 100,000 cheap operations
  and sorting does about 1.7 million. When `k` is close to `n`, `log k` is about `log n`, the heap
  loses its time advantage and uses O(n) space, so sorting is the better choice.
- O(n log n) time and O(n) space are still true as upper bounds, because `k <= n`, but they hide
  what the heap buys you.

## Solution

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();

        for (int i = 0; i < nums.length; i++) {
            if (minHeap.size() >= k) {
                int top = minHeap.peek();
                if (top < nums[i]) {
                    minHeap.poll();
                    minHeap.offer(nums[i]);
                }
            } else {
                minHeap.offer(nums[i]);
            }
        }
        return minHeap.peek();
    }
}
```
`PriorityQueue<Integer>` in Java is a min-heap by default.

## Mistakes and confusions while working through it

| What went wrong | Fix / insight |
|---|---|
| Wrote the answer index as `n - k + 1` | With 0-based arrays it is `n - k`. `n - k + 1` is the 1-based position |
| Said sorting an `int[]` costs O(n) extra space | `Arrays.sort` on primitives is dual-pivot quicksort: O(log n) stack. O(n) is for `Integer[]` and lists |
| Asked whether a min-heap holds values in sorted order | Only the top is guaranteed. The k-th largest is the smallest of the k largest, so the top is all you read |
| Traced the `k = 4` example using a heap of size 2 | Use the `k` from the example. A size-2 heap on that array gives 5, not 4 |
| First code put the fill and replace logic on the wrong sides of `size() >= k`, so the heap never filled and `peek()` returned `null` | The heap fills while `size() < k`. The compare-and-replace rule applies only once it holds `k` numbers |
| An `else` branch offered a smaller number without removing the top, so the heap grew past `k` | A number that is not larger than the top should be ignored, not added |
| Said SC is O(n) for `k = 2` | The heap holds at most `k` numbers, so it is O(k), which is O(1) for `k = 2` |
