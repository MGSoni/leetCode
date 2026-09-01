# Subarray Sum Equals K

**Pattern:** Prefix Sum + HashMap

## Problem
Given an array of integers `nums` and an integer `k`, return the total number of
continuous subarrays whose sum equals `k`.

Example: `nums = [1,2,3], k = 3` → answer is `2` (`[1,2]` and `[3]`).

## Approach
- Brute force is O(n²): try every subarray, accumulating the sum as the window
  extends (not resumming from scratch each time, which would make it O(n³)).
- Key insight: `sum(i+1..j) = prefixSum[j] - prefixSum[i]`. We want this to equal `k`,
  so rearranging: `prefixSum[i] = prefixSum[j] - k`.
- As we scan left to right computing a running prefix sum, at each index `j` we ask:
  "how many earlier prefix sums equal `prefixSum[j] - k`?" — and add that count to `ans`.
- A HashMap (`prefixSum value → count of times seen`) gives O(1) lookup for this.
- **Seed the map with `{0: 1}` before the loop.** This represents the "empty prefix"
  (running total before any elements are walked) and is what lets subarrays that
  start at index 0 be counted correctly.
- **Order matters each iteration:** check the map for `current - k` **first**, then
  insert/update the map with the **current** running total. Checking and logging use
  two different numbers — mixing them up (logging `current - k` instead of `current`)
  breaks the algorithm since future steps can never find the right position.
- Can be done as a true single pass (track `sum` as a running int instead of
  precomputing a full `prefixSum[]` array) to avoid the extra O(n) array — worth
  refactoring to as a follow-up.

## Complexity
- Time: O(n)
- Space: O(n) for the hashmap

## Solution

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int n = nums.length;
        int ans = 0;
        int[] prefixSum = new int[n];
        prefixSum[0] = nums[0];
        for (int i = 1; i < n; i++) {
            prefixSum[i] = prefixSum[i-1] + nums[i];
        }

        Map<Integer, Integer> countMap = new HashMap<>();
        countMap.put(0, 1);

        for (int i : prefixSum) {
            int temp = i - k;

            if (countMap.containsKey(temp)) {
                int value = countMap.get(temp);
                ans += value;
            }

            if (countMap.containsKey(i)) {
                countMap.put(i, countMap.get(i) + 1);
            } else {
                countMap.put(i, 1);
            }
        }
        return ans;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Checked map for `prefixSum[j] - k` but then inserted `prefixSum[j] - k` into the map instead of the actual current prefix sum | Check `current - k` against the map, but always insert `current` (your real position) — the value you check and the value you log are different numbers |
| Proposed rearranging the equation as `prefixSum[i] = k + prefixSum[j]` | Verified by hand-tracing `[1,2,3], k=3` — this rearrangement never finds a match. Correct form is `prefixSum[i] = prefixSum[j] - k`. Always test a proposed algebra rearrangement against a known example before trusting it. |
