# Find Median from Data Stream (LeetCode 295)

## Problem
Design a structure supporting:
- `addNum(num)` — add a number from a continuous stream
- `findMedian()` — return the median of all numbers seen so far

## Core Intuition
Split all seen numbers into two halves:
- **maxHeap** — holds the *smaller* half (top = largest of the small numbers)
- **minHeap** — holds the *larger* half (top = smallest of the large numbers)

The median sits at the boundary between these two heaps, so it's always cheaply reachable via `peek()`.

## Algorithm
**Insert(x):**
- If `x <= maxHeap.peek()` (or maxHeap is empty) → push into `maxHeap`
- Else → push into `minHeap`

**Rebalance** (after every insert):
- If `|maxHeap.size() - minHeap.size()| > 1`, pop from the bigger heap and push into the smaller one

**findMedian():**
- If sizes equal → average of both tops
- Else → top of the bigger heap

## Java Implementation
```java
class MedianFinder {
    PriorityQueue<Integer> minHeap;
    PriorityQueue<Integer> maxHeap;

    public MedianFinder() {
        this.minHeap = new PriorityQueue<>();
        this.maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    }

    public void addNum(int num) {
        if (maxHeap.isEmpty()) {
            maxHeap.add(num);
        } else if (num <= maxHeap.peek()) {
            maxHeap.add(num);
        } else {
            minHeap.add(num);
        }

        if (Math.abs(maxHeap.size() - minHeap.size()) > 1) {
            if (maxHeap.size() > minHeap.size()) {
                int temp = maxHeap.poll();
                minHeap.add(temp);
            } else {
                int temp = minHeap.poll();
                maxHeap.add(temp);
            }
        }
    }

    public double findMedian() {
        if (maxHeap.size() == minHeap.size()) {
            return ((double) maxHeap.peek() + (double) minHeap.peek()) / 2;
        } else if (maxHeap.size() > minHeap.size()) {
            return (double) maxHeap.peek();
        } else {
            return (double) minHeap.peek();
        }
    }
}
```

## Complexity
| Operation | Time | Notes |
|---|---|---|
| `addNum()` | O(log n) | one heap insert + at most one poll+add for rebalancing |
| `findMedian()` | O(1) | just peek() calls |
| Space | O(n) | every inserted number lives in exactly one heap |

## Bugs I hit (watch for these again)
- Declared heaps **inside** the method instead of as class fields → state didn't persist across calls.
- Used `.push()`/`.pop()` (not real `PriorityQueue` methods) instead of `.add()`/`.poll()`.
- Used `Math.Mod()` (not real) instead of `Math.abs()`.
- Cast to `double` *after* integer division was already done: `(double)((a+b)/2)` truncates before casting. Fix: cast operands first — `((double)a + (double)b)/2`.

## Practice link
LeetCode 295: https://leetcode.com/problems/find-median-from-data-stream/
