# Koko Eating Bananas

**Pattern:** Binary search on the answer (the answer is a value, and a check function says whether it works)

## Problem
Koko has `piles[i]` bananas in the i-th pile and `h` hours before the guards return. She picks an
eating speed `k` (bananas per hour). Each hour she picks one pile and eats `k` bananas from it. If
the pile has fewer than `k`, she eats all of it and does not start another pile that hour. Return the
**minimum** integer speed that lets her finish all the bananas within `h` hours.

Example: `piles = [3,6,7,11]`, `h = 8` -> `4`
Example: `piles = [30,11,23,4,20]`, `h = 5` -> `30`
Example: `piles = [30,11,23,4,20]`, `h = 6` -> `23`

LeetCode 875: https://leetcode.com/problems/koko-eating-bananas/
Constraint: `1 <= piles.length <= 10,000`, `piles.length <= h <= 1,000,000,000`,
`1 <= piles[i] <= 1,000,000,000`

## How to recognize it
- The question asks for a **minimum or maximum value** such that some condition holds ("minimum speed
  such that she finishes in time").
- You can write a **check function**: given a candidate value, answer yes or no.
- The check is **monotonic**: if a value works, every larger one works too (or the reverse).
- Trying every value is far too slow (up to 10^9 speeds, and each check goes through every pile).

## Why it is monotonic
"Monotonic" means the values move in one direction only. For `piles = [3,6,7,11]`:

| Speed | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| Total hours | 27 | 15 | 10 | 8 | 8 | 6 |
| Fits in `h = 8`? | no | no | no | yes | yes | yes |

The hours only go down or stay the same as the speed goes up, so the "fits" row switches from no to
yes exactly once: `no no no yes yes yes`. A faster speed never makes any pile take longer.
That lets one test in the middle throw away half the range: a "no" rules out every slower speed, and a
"yes" rules out everything to the right of it. If the answers were `no yes no yes yes no`, a "no" in the
middle would tell you nothing about the left side, and binary search would stop working.

## The three decisions
1. **The range.** `lo = 1` and `hi = max(piles)`.
   - `hi` always works: at a speed equal to the largest pile, every pile takes 1 hour, so the total is
     `piles.length`, and the problem guarantees `piles.length <= h`. A larger speed is never needed.
   - `lo` is **not** the smallest pile. For `piles = [10, 10]` with `h = 20`, speed 1 takes exactly 20
     hours, so the answer is 1, below the smallest pile. Speed 0 never finishes, so 1 is the minimum.
2. **The check.** Total hours at a speed: for each pile, `pile / speed` if `pile % speed == 0`,
   otherwise `pile / speed + 1`. A pile of 7 at speed 3 takes `7/3 + 1 = 3` hours.
   - One-liner with the same result: `(pile + speed - 1) / speed`. Adding `speed - 1` pushes any
     leftover over the next multiple, so it rounds up, and an even division stays put.
   - `Math.ceil(7 / 3)` returns 2, not 3, because `7 / 3` is integer division and drops the remainder
     before `ceil` sees it.
   - The total must be a **`long`**. At speed 1 with 10,000 piles of a billion each it is about 10^13,
     far past an `int`'s limit of about 2.1 x 10^9.
3. **The move.**
   - If `mid` works: the answer is `mid` or something smaller. **`mid` stays in**, so `hi = mid`.
   - If `mid` fails: `mid` and everything slower fail, so `lo = mid + 1`.

## The loop template
```
lo = 1, hi = max(piles)
while (lo < hi):
    mid = lo + (hi - lo) / 2
    if hours(mid) <= h: hi = mid
    else:               lo = mid + 1
return lo
```
- **Why `lo < hi` and not `lo <= hi`:** with `hi = mid` the range can stop shrinking. If `lo == hi == 4`
  and `mid` works, `hi = mid` leaves `hi` at 4, so a `<=` loop never ends. With `lo < hi`, the loop stops
  when they meet, and that value is the answer.
- **Why the answer is the smallest working speed:** `hi` always works, because it only moves to a `mid`
  that was tested and worked, and it starts at the largest pile. Everything below `lo` always fails,
  because `lo` only moves to `mid + 1` after `mid` failed. When they meet, that value works and nothing
  below it does.
- **Midpoint:** `lo + (hi - lo) / 2` gives the same value as `(lo + hi) / 2` and cannot overflow. Integer
  division rounds toward `lo`, which matches `hi = mid` and `lo = mid + 1`.
- **Fits within `h`** means total hours `<= h`, not `< h`.

## Hand traces
**`[3,6,7,11]`, `h = 8`:**

| lo | hi | mid | Total hours | Branch |
|---|---|---|---|---|
| 1 | 11 | 6 | 6 | works, `hi = 6` |
| 1 | 6 | 3 | 10 | fails, `lo = 4` |
| 4 | 6 | 5 | 8 | works, `hi = 5` |
| 4 | 5 | 4 | 8 | works, `hi = 4` |

`lo == hi == 4`, so the answer is 4.

**`[30,11,23,4,20]`, `h = 5`:**

| lo | hi | mid | Total hours | Branch |
|---|---|---|---|---|
| 1 | 30 | 15 | 8 | fails, `lo = 16` |
| 16 | 30 | 23 | 6 | fails, `lo = 24` |
| 24 | 30 | 27 | 6 | fails, `lo = 28` |
| 28 | 30 | 29 | 6 | fails, `lo = 30` |

`lo == hi == 30`, so the answer is 30. Every speed below 30 needs at least 6 hours, because the pile of
30 takes 2 hours unless the speed is at least 30.

## Complexity
- **Time: O(n log M)**, where `n` is the number of piles and `M` is the largest pile. The range `1..M`
  halves each time, so the search tries about `log2(M)` speeds, and each try goes through all `n` piles.
  With `M` = 1 billion and `n` = 10,000, that is about 30 tries x 10,000 piles = 300,000 steps. Trying
  every speed from 1 up would take about 10^13 steps.
- **Space: O(1).** A few variables.

## Solution

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int high = Integer.MIN_VALUE;
        int low = 1;

        for (int i : piles) {
            if (i > high) {
                high = i;
            }
        }
        while (low < high) {
            int mid = low + (high - low) / 2;
            long time = hoursTaken(piles, mid);
            if (time <= h) {
                high = mid;
            } else {
                low = mid + 1;
            }
        }
        return low;
    }

    private long hoursTaken(int[] piles, int speed) {
        long hours = 0;
        for (int i : piles) {
            if (i % speed == 0) {
                hours += i / speed;
            } else {
                hours += (i / speed) + 1;
            }
        }
        return hours;
    }
}
```

## Same pattern, other problems
- Capacity to Ship Packages Within D Days (LC 1011): same loop, different check function.
- Split Array Largest Sum (LC 410): the same idea, with a "minimize the maximum" check.

## Mistakes and confusions while working through it

| What went wrong | Fix / insight |
|---|---|
| Proposed the lower end of the range as the smallest pile | The answer can be below the smallest pile (`[10, 10]`, `h = 20` gives 1). The lowest possible speed is 1 |
| Did not see why `lo <= hi` is a problem when `mid` works | With `hi = mid` the range can stop shrinking, so a `<=` loop never ends. Use `lo < hi` and return `lo` |
| Said "if mid works, keep the left half" without saying whether `mid` stays in | `mid` stays in, so `hi = mid`, not `mid - 1`. Dropping it could discard the answer |
| Described the overflow as "more than a million" | The total can reach about 10^13. The thing to compare with is an `int`'s limit of about 2.1 x 10^9 |
| First code wrote `hours += i % h` in the even-division branch | An even division adds `i / speed`. `%` gives the remainder, not the hours |
| First code used `time < h` | Finishing within `h` hours means `<= h`. Speed 4 takes exactly 8 hours on the first example |
| First code kept the hours total as an `int` after saying it needed a `long` | Change the variable and the helper's return type together |
| Named the helper's speed parameter `h`, which looks like the hours limit | Name it `speed` |
| Gave TC as "log of the highest pile" | That is the number of tries. Each try costs O(n), so the total is O(n log M) |
