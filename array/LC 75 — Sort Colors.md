# LeetCode 75 — Sort Colors

**Pattern:** Three-pointer partitioning (Dutch National Flag)
**Difficulty:** Medium
**Link:** https://leetcode.com/problems/sort-colors/

---

## Problem
Given an array containing only 0s, 1s, and 2s, sort it in-place in a single pass, without using a library sort.

---

## My Solution (revision attempt — correct)
```java
class Solution {
    public void sortColors(int[] nums) {
        int zero = 0;
        int one = 0;
        int two = nums.length-1;

        while(one<=two){
            if(nums[one]==2){
                int temp = nums[two];
                nums[two] = nums[one];
                nums[one] = temp;
                two--;
            }else if(nums[one]==0){
                int temp = nums[zero];
                nums[zero] = nums[one];
                nums[one] = temp;
                zero++;
                one++;
            }else{
                one++;
            }
        }
    }
}
```
**Verdict:** ✅ Correct. `zero/one/two` are the same as `low/mid/high` from the standard derivation.
**Time:** O(n) — single pass, each pointer collectively traverses the array once.
**Space:** O(1) — in-place, only pointer variables.

---

## Core Insight (Three-Pointer Partitioning)
- **3 categories to partition → 3 pointers**, not 2. Two pointers can only mark two boundaries; a third "scanning" pointer (`one`/`mid`) is needed to represent "the next unexamined element."
- Invariant: before `zero` → confirmed 0s | `zero` to `one` → confirmed 1s | `one` to `two` → unexamined | after `two` → confirmed 2s
- **Why `one` advances after a 0-swap but not after a 2-swap:**
  - Swap with `zero`: the value swapped back into position `one` came from the **already-confirmed 0s/1s region** (between `zero` and `one`) — guaranteed to be a 1, safe to move past.
  - Swap with `two`: the value swapped back into position `one` came from the **unexamined region** near `two` — could be anything (0, 1, or 2), must re-check before advancing.

---

## Full Trace: `[2,0,2,1,1,0]`
| one | nums[one] | action | zero | two | array |
|---|---|---|---|---|---|
| 0 | 2 | swap(one,two), two-- | 0 | 4 | [0,0,2,1,1,2] |
| 0 | 0 | swap(zero,one), zero++, one++ | 1 | 4 | [0,0,2,1,1,2] |
| 1 | 0 | swap(zero,one), zero++, one++ | 2 | 4 | [0,0,2,1,1,2] |
| 2 | 2 | swap(one,two), two-- | 2 | 3 | [0,0,1,1,2,2] |
| 2 | 1 | one++ | 2 | 3 | [0,0,1,1,2,2] |
| 3 | 1 | one++ | 2 | 3 | [0,0,1,1,2,2] |

Loop ends: `one=4 > two=3`. Final: `[0,0,1,1,2,2]` ✓

---

## Mistakes / Things to Remember
- A `for` loop doesn't fit naturally here since the scanning pointer's advancement is conditional (not a flat +1 every iteration) — a `while` loop (or `for` with empty increment clause) is the honest structure.
- When `low == mid` (early in the loop, e.g. at start), the swap is a no-op (swapping a value with itself) — harmless, not a bug; the logic is written generically to handle both `low == mid` and `low < mid` with the same line.
- General rule: if a problem has **N categories** to partition into, expect to need **N pointers** — one per boundary, plus a scanner for the unexamined middle.

## Related Problems
- Quicksort's 3-way partition (same Dutch National Flag idea, applied as a subroutine)
- Partition Array by categories (general pattern for any "sort into exactly K buckets in one pass")
