# Greedy — 435. Non-overlapping Intervals

## Problem

Given a list of intervals, find the minimum number of intervals to remove
so that the rest are non-overlapping.

Equivalent reframe (easier to solve):

> Find the MAXIMUM number of non-overlapping intervals you can keep.
> Answer = total - that max.

---

## Why sort by END time (not start time)

This is the core greedy insight for interval scheduling.

```
Sorting by START time:
  a long early interval can block out many later ones
  → doesn't help you decide what to keep

Sorting by END time:
  the interval that finishes earliest leaves the MOST room
  for future intervals
  → always greedily keep the one that ends soonest
```

**The question to ask on any interval problem:**

> Am I trying to fit as many non-overlapping intervals as possible?
> → sort by END time, greedily keep the earliest-ending one that fits.

---

## Walkthrough of the logic

```
1. Sort intervals by end time.
2. Always "keep" the first interval (nothing to conflict with yet).
3. For every next interval:
     if its start >= lastEnd  → no overlap → keep it, update lastEnd
     else                     → it overlaps → skip it (this is the one we'd remove)
4. count = number of intervals kept
5. answer = total - count  (intervals removed)
```

---

## Code

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by end time

        int total = intervals.length;
        int lastEnd = intervals[0][1];
        int count = 1; // first interval is always kept

        for (int i = 1; i < total; i++) {
            if (intervals[i][0] >= lastEnd) {
                // no overlap → keep this interval
                lastEnd = intervals[i][1];
                count++;
            }
            // else: overlaps → skip it, lastEnd stays the same
        }

        return total - count;
    }
}
```

---

## Complexity

```
Time:  O(n log n) — dominated by the sort
Space: O(1) extra (excluding sort's internal space)
```

---

## Mistakes to watch for on this pattern

- **Sorting by start instead of end** → breaks the greedy proof, gives wrong answer on some inputs.
- **Off-by-one on overlap check** → `>=` vs `>`. Touching intervals like `[1,2]` and `[2,3]` do NOT overlap, so it must be `intervals[i][0] >= lastEnd`, not `>`.
- **Forgetting `count starts at 1`** → the first interval (after sorting) is always kept since nothing came before it.
- **Returning `count` instead of `total - count`** → the question asks for removals, not survivors.

---

## Generalize: the "interval scheduling" greedy template

Use this template whenever you see:
"maximum number of non-overlapping intervals" / "minimum removals to make non-overlapping" / "maximum number of meetings/events you can attend."

```
1. Sort by END time.
2. Track lastEnd = end of last kept interval.
3. Iterate:
     start >= lastEnd → keep, update lastEnd
     start <  lastEnd → discard (overlap)
4. Derive the final answer from count kept vs total.
```
