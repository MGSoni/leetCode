# LeetCode 121 — Best Time to Buy and Sell Stock

**Pattern:** Running-value single pass (min-tracking)
**Difficulty:** Easy
**Link:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock/

---

## Problem
Given an array `prices` where `prices[i]` is the stock price on day `i`, find the maximum profit from buying on one day and selling on a later day. Return 0 if no profit is possible.

---

## My Solution
```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfit = 0;
        int minSoFar = Integer.MAX_VALUE;
        for(int price: prices){
            if(price < minSoFar){
                minSoFar = price;
            }
            else if(price-minSoFar > maxProfit){
                maxProfit = price - minSoFar;
            }
        }
        return maxProfit;
    }
}
```

**Verdict:** ✅ Correct. O(n) time, O(1) space.

**Why the `else if` is safe here (not a bug):** on the day you find a new minimum, checking `price - minSoFar` using the *old* `minSoFar` would give a value ≤ 0 anyway (since `price` is lower than the old min), so it could never beat an existing `maxProfit` ≥ 0. Skipping the profit check on that specific day loses nothing. (Contrast with the two-pointer attempt on this same problem, which had a real bug, not just a stylistic difference — see concept card.)

---

## Other Approaches Explored
1. Brute force — O(n²), check every (buy, sell) pair
2. Min-tracking single pass (this solution) — O(n) / O(1)
3. Reverse scan, max-tracking from the right — O(n) / O(1)
4. Kadane's on day-to-day price differences — O(n) / O(1)
5. State machine (hold/cash) — O(n) / O(1), generalizes to Stock II/III/IV
6. Divide and conquer — O(n log n) / O(log n)
7. Two-pointer (converging) — ❌ attempted, proven incorrect (fails on `[3,2,6,5,0,3]`)

---

## Mistakes / Things to Remember
- Two-pointer convergence needs a provable elimination argument — this problem doesn't have one, unlike Container With Most Water.
- Initialize `minSoFar` to `Integer.MAX_VALUE` (or `prices[0]`), never 0.

## Related Problems
- Best Time to Buy and Sell Stock II (122) — greedy, multiple transactions allowed
- Best Time to Buy and Sell Stock III (123) — DP, at most 2 transactions
- Best Time to Buy and Sell Stock IV (188) — DP, at most k transactions
- Best Time to Buy and Sell Stock with Cooldown (309)
- Best Time to Buy and Sell Stock with Transaction Fee (714)
