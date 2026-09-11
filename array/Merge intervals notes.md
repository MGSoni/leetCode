# Merge Intervals — Study Notes

**Pattern:** Interval Merging (shows up heavily in booking/reservation-style system design & coding rounds — e.g., Airbnb)

**LeetCode:** [56 - Merge Intervals](https://leetcode.com/problems/merge-intervals/)

---

## Problem
Given a list of intervals `[start, end]`, merge all overlapping intervals and return the non-overlapping result.

**Example:**
Input: `[[1,3], [2,6], [8,10], [15,18]]`
Output: `[[1,6], [8,10], [15,18]]`

---

## Approach
1. **Sort intervals by start time.** This guarantees any overlap only ever needs to be checked against the *most recently merged* interval — never further back.
2. **Walk through once**, keeping a `current` merged interval:
   - If the next interval's start `<= current`'s end → they overlap (or touch) → extend `current`'s end.
   - Otherwise → gap found → close `current`, start a new one.

### Why `interval[0] <= current[1]` means overlap
Since sorted by start time, if the next interval starts before (or exactly when) the current one ends, they share space on the number line. If there's a gap between current's end and the next interval's start, there's no overlap.

---

## Code (Java)

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        List<int[]> result = new ArrayList<>();
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start time

        int[] current = intervals[0];
        result.add(current);

        for (int[] interval : intervals) {
            if (interval[0] <= current[1]) {
                current[1] = Math.max(interval[1], current[1]); // extend merge
            } else {
                current = interval;
                result.add(current);
            }
        }

        return result.toArray(new int[result.size()][]);
    }
}
```

---

## Key Insight: Reference Semantics (important — easy to miss)

`current` and `result`'s entry are the **same array object in memory**, not copies.

```java
int[] current = intervals[0];  // current points to same array as intervals[0]
result.add(current);            // result's entry is a reference to that same array
```

When you later do `current[1] = ...`, you're **mutating** that shared array — so `result` reflects the change automatically, without ever calling `result.add()` again for that entry. This is why `result.add(current)` only appears twice in the whole method: once at start, once when a genuinely new interval begins (the `else` branch).

**Good line to say in an interview:**
*"Since `current` and the entry in `result` are the same array reference, mutating `current[1]` updates it in place — so `result` reflects the merge automatically."*

---

## Complexity

| | Complexity | Why |
|---|---|---|
| **Time** | O(n log n) | Sort dominates (O(n log n)); the loop and array conversion are both O(n), which get dominated by the sort term. |
| **Space** | O(n) | The `result` list holds up to n intervals in the worst case (no overlaps at all) — this counts as space even though sorting itself is in-place (O(1) auxiliary). |

**How to say it:** *"O(n log n) time, O(n) space for the output — O(1) auxiliary space beyond that, since the sort is in-place."*

---

## Related Problems to Know (same pattern family)
- **Insert Interval** (LC 57) — insert + merge a new interval into an already-merged list
- **Meeting Rooms II** (LC 253) — find minimum resources/rooms needed given overlapping intervals (two-pointer on separated start/end arrays)
- **Non-overlapping Intervals** (LC 435) — minimum removals to make all intervals non-overlapping

---

## Common Mistakes to Avoid
- Forgetting to sort first — the whole single-pass logic depends on sorted order.
- Manually writing a bubble/selection sort instead of `Arrays.sort()` — signals you don't know the standard library; always use the built-in sort in an interview.
- Saying space is O(1) without accounting for the output list.
