# Contiguous Array

**Pattern:** Prefix Sum + HashMap (first-occurrence index)

## Problem
Given a binary array `nums` (only `0`s and `1`s), return the maximum length of a
contiguous subarray with an equal number of `0`s and `1`s.

Example: `nums = [0,1]` → `2`

LeetCode 525: https://leetcode.com/problems/contiguous-array/

## Core idea — turn "equal 0s and 1s" into "sum equals 0"
Treat every `0` as `-1` and every `1` as `+1`. A subarray has an equal number of
0s and 1s exactly when its transformed sum is `0`.

So the problem becomes: **find the longest subarray with sum 0**, which is the
same shape as [[subarray-sum-equals-k]] with `k = 0`.

Using prefix sums: if `prefix[i] == prefix[j]` for some `j < i`, then the
subarray `(j+1 .. i)` sums to 0, and its length is `i - j`.

## Key difference from Subarray Sum = K
| | Subarray Sum = K | Contiguous Array |
|---|---|---|
| Question | **How many** subarrays? | **Longest** subarray? |
| Map stores | prefix sum → **count** of times seen | prefix sum → **first index** where seen |
| On a repeat | add the count to the answer | compute `i - firstIndex`, keep the max |
| Update rule | always increment the count | **only put if absent** (never overwrite) |

**Why only the first occurrence:** the earliest index for a given prefix sum gives
the longest possible subarray ending at `i`. Overwriting it with a later index
would shorten the result.

## Seed the map with `{0: -1}`
`-1` represents "the empty prefix before index 0". Without it, a valid subarray
starting at index 0 (for example `[0,1]`, where the prefix sum becomes `0` at
index 1) would never be found. With it, the length is `1 - (-1) = 2`.
This is the same role the `{0: 1}` seed played in Subarray Sum = K.

## Algorithm
1. Build the running sum, adding `-1` for a `0` and `+1` for a `1`.
2. Map `{0: -1}` to start.
3. For each index `i` with running sum `s`:
   - If `s` is already in the map: `result = max(result, i - map.get(s))`
   - Otherwise: `map.put(s, i)` (record first occurrence only)
4. Return `result`.

## Hand trace — `nums = [0,1,0]`
- Transformed prefix sums: `-1, 0, -1`
- Map starts `{0: -1}`
- `i=0`, sum `-1`: not in map → put `(-1, 0)`
- `i=1`, sum `0`: in map at `-1` → length `1 - (-1) = 2`, result = 2
- `i=2`, sum `-1`: in map at `0` → length `2 - 0 = 2`, result stays 2
- Answer: `2` ✓

## Complexity
- **Time: O(n)** — one pass to build prefix sums, one pass over the map lookups,
  each lookup O(1).
- **Space: O(n)** — the `prefixArr` array is O(n) and the map holds up to O(n)
  distinct prefix sums.

(Verify these match what you derived yourself — I filled them in from the code.)

## Follow-up optimization
The `prefixArr` array isn't needed. Keep a single running `sum` variable and do
everything in one loop. That drops the extra array and makes it a true single
pass, with the same O(n) time. Space stays O(n) because of the map, but you avoid
the second O(n) structure.

## Solution (as submitted)

```java
class Solution {
    public int findMaxLength(int[] nums) {
        int n = nums.length;
        int[] prefixArr = new int[n];
        int sum = 0;
        for (int i = 0; i < n; i++) {
            if (nums[i] == 0) {
                sum += -1;
            } else {
                sum += nums[i];
            }
            prefixArr[i] = sum;
        }

        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1);
        int result = 0;
        for (int i = 0; i < n; i++) {
            int key = prefixArr[i];
            if (map.containsKey(key)) {
                int index = map.get(key);
                result = Math.max(result, i - index);
            } else {
                map.put(key, i);
            }
        }
        return result;
    }
}
```

## Single-pass version (for revision)

```java
class Solution {
    public int findMaxLength(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1);
        int sum = 0;
        int result = 0;
        for (int i = 0; i < nums.length; i++) {
            sum += (nums[i] == 0) ? -1 : 1;
            if (map.containsKey(sum)) {
                result = Math.max(result, i - map.get(sum));
            } else {
                map.put(sum, i);
            }
        }
        return result;
    }
}
```

## Mistake log
No bugs were reported on this one. Add a row here if you hit anything while
revising:

| Pattern | What went wrong | Fix / insight |
|---|---|---|
| | | |
