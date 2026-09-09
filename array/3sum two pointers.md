# 3Sum

**Pattern:** Sorting + Two Pointers

## Problem
Given an integer array `nums`, return all unique triplets `[a, b, c]` such that
`a + b + c == 0`. The solution set must not contain duplicate triplets.

## Approach
- Sort the array first — this enables both the two-pointer technique and easy
  duplicate-skipping.
- Fix one element at index `i` (outer loop), then use two pointers (`left = i+1`,
  `right = nums.length-1`) to search the remaining subarray for a pair that sums
  to `-nums[i]`.
- **Early exit:** if `nums[i] > 0`, break entirely — since the array is sorted,
  once the smallest remaining value is positive, no triplet can sum to zero.
- **Skip duplicate `i` values:** `if (i > 0 && nums[i] == nums[i-1]) continue;`
  — this must NOT gate whether the two-pointer search runs at all (i=0 always
  needs to run); it only skips reprocessing an identical `i`.
- Two-pointer loop logic:
  - If `sum == 0`: record the triplet, then skip past any duplicate values at
    `left` and `right` *before* moving both pointers inward (skip duplicates
    first, then step past the just-used value).
  - If `sum < 0`: move `left++` (need a larger sum).
  - If `sum > 0`: move `right--` (need a smaller sum).
  - Every branch must move at least one pointer, or the loop never terminates.

## Complexity
- **Time: O(n²)** — `Arrays.sort()` is O(n log n), outer loop is O(n), inner
  two-pointer loop is O(n) per outer iteration → O(n log n) + O(n²), dominated
  by O(n²).
- **Space: O(log n)** — no extra array is used, but `Arrays.sort()` on a
  primitive `int[]` uses dual-pivot quicksort, which is in-place but uses
  O(log n) recursion stack space in the average case. (Note: sorting an array
  of objects like `Integer[]` would use TimSort instead, which is O(n) space —
  the primitive-vs-object distinction matters.)

## Solution

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < nums.length - 2; i++) {
            if (nums[i] > 0) break;

            if (i > 0 && nums[i] == nums[i-1]) {
                continue;
            }

            int left = i + 1;
            int right = nums.length - 1;
            while (left < right) {
                int sum = nums[left] + nums[right] + nums[i];
                if (sum == 0) {
                    List<Integer> temp = new ArrayList<>();
                    temp = List.of(nums[i], nums[left], nums[right]);
                    result.add(temp);
                    while (left < right && nums[left] == nums[left+1]) left++;
                    while (left < right && nums[right] == nums[right-1]) right--;
                    left++;
                    right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return result;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Wrapped the entire two-pointer search inside `if(i>0)`, which would skip the valid i=0 case entirely | The duplicate-skip check (`nums[i]==nums[i-1]`) and "should this iteration run at all" are different concerns — don't conflate them |
| `left` pointer initialized to `i-1` instead of `i+1` | `left` must start strictly after `i`, since the two-pointer search covers only the remaining unfixed portion of the array |
| No pointer movement inside the while loop in any branch — infinite loop risk | Every branch of the sum comparison (`==0`, `<0`, `>0`) must move at least one pointer, or the loop never terminates |
| Assumed O(1) space since no explicit extra array was used | `Arrays.sort()` on primitives uses dual-pivot quicksort — in-place but with O(log n) recursion stack space; don't forget to account for what the standard library call itself costs |
