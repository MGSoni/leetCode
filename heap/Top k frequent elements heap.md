# Top K Frequent Elements

**Pattern:** Hashing (frequency count) + heap, top-K ordered by a custom key

## Problem
Given an integer array `nums` and an integer `k`, return the `k` most frequent elements, in any
order. The answer is guaranteed to be unique.

Example: `nums = [1,1,1,2,2,3]`, `k = 2` -> `[1,2]`
Example: `nums = [1]`, `k = 1` -> `[1]`

LeetCode 347: https://leetcode.com/problems/top-k-frequent-elements/
Constraint: `1 <= nums.length <= 100,000`. `k` is between 1 and the number of distinct elements.

## Core insight
This is Kth Largest ([[kth-largest-element-min-heap]]) with one change: the heap is ordered by
**frequency**, not by the number itself.
1. One pass builds a frequency map: `{1:3, 2:2, 3:1}`.
2. Keep a **min-heap of size `k` ordered by frequency**. The top is the least frequent of the k
   most frequent numbers. When the heap grows past `k`, remove the top.
3. What is left in the heap is the answer.

**A map does not come out in frequency order.** Iterating a `HashMap` and taking the first `k`
entries gives the wrong numbers. The heap is what does the ordering.

## The comparator
`new PriorityQueue<Integer>()` orders by the number itself. To order by frequency, pass a
comparator:
```java
new PriorityQueue<Integer>((a, b) -> freq.get(a) - freq.get(b));
```
It takes two numbers and returns a negative value if `a` should be closer to the top, positive if
`b` should, and zero if it does not matter. `freq.get(a) - freq.get(b)` puts the **lowest
frequency on top** (a min-heap by frequency). Swapping the terms flips it.

## Approaches
| Approach | Time | Space | Notes |
|---|---|---|---|
| Sort the distinct entries by frequency, take `k` | O(n + d log d) | O(d) | Simple and fine at this constraint |
| Max-heap of all distinct entries, poll `k` times | O(n + d log d) | O(d) | Same cost as sorting |
| **Min-heap of size `k`** (used here) | O(n + d log k) | O(d) for the map, O(k) for the heap | Best when `k` is small |
| Bucket sort by frequency | O(n) | O(n) | Index = frequency, since a count can't exceed `n`. Follow-up, not practiced yet |

`n` = array length, `d` = number of distinct values, `k` = how many to keep.

## Algorithm
```
build freq map                                  // O(n)
for each distinct number in freq.keySet():      // d iterations, not n
    heap.offer(number)
    if heap.size() > k: heap.poll()             // drops the least frequent
poll the heap k times into result[]
```
The loop goes over the **distinct** numbers. Looping over `nums` visits a repeated number once
for each time it appears, so the same number can end up in the heap more than once.

## Hand trace
`[1,1,1,2,2,3]`, `k = 2`, map `{1:3, 2:2, 3:1}`:
- Offer 1: heap `{1}`.
- Offer 2: heap `{2, 1}` (2 on top, frequency 2 is lower than 3).
- Offer 3: heap `{3, 2, 1}`, size 3 is more than `k`, so poll the top, which is 3 (frequency 1).
- Final heap `{2, 1}`. Polling gives `2` then `1`, so `result = [2, 1]`. Any order is accepted.

## Complexity
- **Time: O(n + d log k).** Counting is O(n). The heap loop offers `d` numbers, and each offer or
  poll costs O(log k), because the heap never holds more than `k + 1` numbers. Since `d <= n`,
  the worst case is O(n log k). O(n log n) is still true as an upper bound (`k <= n`), but it
  hides the `log k`.
- **Space: O(d), which is O(n) in the worst case.** The map holds `d` entries. The heap holds at
  most `k + 1` numbers and `result` holds `k`, and both are at most `d`.

## Solution

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int x : nums) {
            freq.put(x, freq.getOrDefault(x, 0) + 1);
        }

        PriorityQueue<Integer> minHeap = new PriorityQueue<>((a, b) -> freq.get(a) - freq.get(b));
        for (int num : freq.keySet()) {
            minHeap.offer(num);
            if (minHeap.size() > k) {
                minHeap.poll();
            }
        }

        int[] result = new int[k];
        for (int i = 0; i < k; i++) {
            result[i] = minHeap.poll();
        }
        return result;
    }
}
```
`getOrDefault(x, 0) + 1` reads the current count (0 if the number is new) and writes back one more.

## Mistakes and confusions while working through it

| What went wrong | Fix / insight |
|---|---|
| Planned to iterate the map and take the first `k` values | A map is not in frequency order. Order the entries with a heap |
| `int count = map.get(n); ++count;` without writing it back | `count` is a local copy. Call `map.put(n, count)` (or use `getOrDefault`) |
| Used `new PriorityQueue<>()` with no comparator | It orders by the number, so `peek()` returned the smallest number, not the least frequent. Pass `(a, b) -> freq.get(a) - freq.get(b)` |
| Filled the heap by looping over `nums` | A repeated number was offered several times. Loop over the distinct keys, `freq.keySet()` |
| `result.toArray(new int[...])` on a `List<Integer>` | Does not compile, because `toArray` needs a reference-type array. Fill an `int[]` directly |
| Wrote `nums[i] = minHeap.poll()` instead of `result[i] = ...` | Check which array you are filling and which one you return |
| Gave TC as O(n log n) | True as an upper bound, but the tight answer is O(n + d log k). The heap stays at size `k`, so each operation costs `log k` |
