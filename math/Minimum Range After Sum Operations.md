# Minimum Range After Sum Operations (observation / math)

**Source:** LeetCode Bi-weekly Contest 196, Q2 (Medium). Link: https://leetcode.com/contest/biweekly-contest-196/problems/minimum-range-after-sum-operations/

**Problem:** Given three integers `a`, `b`, `c` (each 1 to 100). One operation: pick one of the three and replace it with the sum of the other two. The range is `max - min`. Return the minimum range reachable after any number of operations (including zero). (The statement also asks you to name a variable `lomirevatu` to store the input midway. That is contest noise and doesn't change the logic.)

**Pattern:** Small constraints plus a simple operation means: work examples by hand and ask "what does each operation do to the range, and when can it ever help?" No data structure needed.

---

## Core insight

All numbers are positive, so replacing a number with the sum of the other two makes it **bigger than both of them**. That decides which replacement can ever help:

| Replace the... | New array | Effect on range |
|---|---|---|
| smallest | it becomes the new max | range only grows or stays. Never useful |
| middle | it becomes the new max | range only grows or stays. Never useful |
| **biggest** | it drops to the sum of the other two | **helps only if the biggest is larger than that sum** |

Sort the three numbers as `x <= y <= z`.

- If `z > x + y`: replace `z` with `x + y`. The numbers are now `x, y, x + y`, with max `x + y` and min `x`, so the range is `(x + y) - x = y`. One replacement is all you ever do, because the new biggest equals the sum of the other two, not more than it, so no further operation helps.
- Otherwise: no operation helps, and the answer is `z - x`.

```
sorted:  x   y   z

z > x + y  →  answer = y         (replace z with x+y once, then stop)
else       →  answer = z - x     (leave as is)
```

## Hand trace

| Input | Sorted (x, y, z) | Is z > x + y? | Answer |
|---|---|---|---|
| 9, 2, 5 | 2, 5, 9 | 9 > 7, yes | y = 5 |
| 5, 7, 6 | 5, 6, 7 | 7 > 11, no | 7 - 5 = 2 |
| 3, 3, 3 | 3, 3, 3 | 3 > 6, no | 3 - 3 = 0 |

Example 1 step by step: replace 9 with 2 + 5 = 7, giving `7, 2, 5` with range 5. Trying each further operation gives `7, 12, 5` (range 7) or `7, 2, 9` (range 7), or no change. So 5 is the minimum. ✓

## Complexity

- **Time: O(1).** Sorting three numbers is constant work.
- **Space: O(1).**

## Code

This one was solved by reasoning on the call. The code below is a reference written afterwards (the method name in the LeetCode stub may differ). It was checked against a brute force over all triples with values 1 to 12, with no mismatches.

```java
import java.util.Arrays;

class Solution {
    public int minimumRange(int a, int b, int c) {
        int[] arr = {a, b, c};
        Arrays.sort(arr);                 // x <= y <= z
        int x = arr[0], y = arr[1], z = arr[2];

        if (z > x + y) {
            return y;                     // replace z with x + y once, range becomes y
        }
        return z - x;                     // no replacement helps
    }
}
```

## Mistakes and notes

| Item | What happened | Fix / rule |
|---|---|---|
| Which number to replace | Wasn't sure which one is worth replacing. Thought replacing the biggest could still make it bigger | The replaced number becomes the sum of the other two. Compare the biggest against that sum: if the biggest is larger, replacing it shrinks the range |
| Reading the operation | Tried operations on 7 and 2, and 2 and 5, without a clear rule | For each option ask: does the new number become the new max? If yes, the range only grows |
| Range after replacing | Needed a nudge to see the range after replacing | After replacing `z`: numbers are `x, y, x+y`, so range = `(x+y) - x = y` |
| When to stop | Wasn't obvious that one replacement is enough | After replacing, the biggest equals the sum of the other two, not more, so the rule says stop |

## Lesson for contests

When the constraints are tiny (here, 100) and the operation is "replace with a sum", **try the examples by hand** and ask what each operation does to the thing you're measuring. The answer here came from one question: "which replacement can ever make the range smaller?"
