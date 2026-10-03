# LeetCode 523 — Continuous Subarray Sum

**Pattern:** Prefix Sum + Hashmap (remainder lookup) — variant of the complement-lookup pattern from Subarray Sum Equals K (560), but storing remainders and first-index instead of raw sums and counts
**Difficulty:** Medium
**Link:** https://leetcode.com/problems/continuous-subarray-sum/

---

## Problem
Given array `nums` and integer `k`, return `true` if there exists a contiguous subarray of length **at least 2** whose sum is a multiple of `k`.

---

## My Solution (cleaned up)
```java
class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        int n = nums.length;
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1); // virtual prefix at index -1, handles subarrays starting at index 0

        int sum = 0;
        for (int i = 0; i < n; i++) {
            sum += nums[i];
            int remainder = sum % k;

            if (map.containsKey(remainder)) {
                int firstIndex = map.get(remainder);
                if (i - firstIndex >= 2) {
                    return true;
                }
            } else {
                map.put(remainder, i); // only store first occurrence
            }
        }
        return false;
    }
}
```
**Time:** O(n), **Space:** O(min(n, k))

---

## Intuition
- Two prefix sums with the **same remainder when divided by k** means their difference is a multiple of k (not necessarily 0 — e.g. remainder-5 values 23 and 29 differ by 6, a multiple of 6, not by 0).
- `prefixSum[j] - prefixSum[i]` = sum of subarray `(i+1 to j)`. If this is a multiple of k, then `prefixSum[i]` and `prefixSum[j]` share the same remainder.
- Map stores `remainder → first index where it occurred` — using the **first** occurrence maximizes the gap to any later match, giving the best shot at satisfying length ≥ 2.
- Seed `map.put(0, -1)`: handles the case where `prefixSum[j]` itself is already a multiple of k (remainder 0) — pretend there's a virtual prefix of 0 at index -1, so `j - (-1) = j+1 ≥ 2` whenever `j ≥ 1`.

---

## Mistakes / Things to Remember
- **Redundant special-casing on first loop iteration:** original draft had an `if(i==0) {...} else {...}` for computing the first prefix value, but `sum` is already correctly updated by the time that line runs — the general-case formula (`sum += nums[i]; prefixArray[i] = sum % k;`) already works correctly at `i=0` without a special case. **Lesson: before adding an `if` for "first iteration," check whether the general-case line already produces the correct result there.**
- Store **first** index per remainder, not every occurrence — using a later occurrence would shrink the gap and could cause false negatives on the length-≥2 check.
- Don't confuse "same remainder" with "difference is 0" — the difference is a *multiple* of k, which could be any multiple, not just 0.
- Edge case worth testing: `nums=[0,0], k=1` — confirms the `-1` seeding correctly handles a subarray starting at index 0.

## Related Problems
- Subarray Sum Equals K (560) — same prefix-sum-complement shape, but counting exact-sum matches instead of remainder matches
- Subarray Sums Divisible by K (974) — same remainder idea, but counting all pairs instead of early-exiting on first match
- Contiguous Array (525) — treat 0s as -1, reduces to "subarray sum equals 0"
