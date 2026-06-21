# 3969. Valid Subarrays With Matching Sum Digits I

## Problem

You are given an integer array `nums` and an integer digit `x`.

A subarray `nums[l..r]` is considered valid if the sum of its elements satisfies both of the following conditions:

* The first digit of the sum is equal to `x`.
* The last digit of the sum is equal to `x`.

Return the number of valid subarrays.

### Example 1

Input:

```text
nums = [1,100,1], x = 1
```

Output:

```text
4
```

Explanation:

The valid subarrays are:

```text
nums[0..0] = 1
nums[0..1] = 101
nums[1..2] = 101
nums[2..2] = 1
```

### Example 2

Input:

```text
nums = [1], x = 2
```

Output:

```text
0
```

### Constraints

```text
1 <= nums.length <= 1500
1 <= nums[i] <= 10^9
1 <= x <= 9
```

---

## Pattern Recognition

The constraints immediately suggest:

```text
n ≤ 1500
```

A brute force solution checking every subarray is feasible.

Number of subarrays:

```text
n * (n + 1) / 2
≈ 1.1 million when n = 1500
```

which is acceptable.

---

## Approach

For every starting index:

```java
i
```

expand the ending index:

```java
j
```

and maintain the running sum.

For each subarray:

```java
nums[i...j]
```

check:

1. Last digit of sum

```java
sum % 10 == x
```

2. First digit of sum

Keep dividing by 10 until only one digit remains.

```java
while(sum >= 10){
    sum /= 10;
}
```

Then verify:

```java
firstDigit == x
```

If both conditions are satisfied:

```java
result++
```

---

## Why Running Sum?

Instead of recomputing the sum for every subarray:

```text
O(n³)
```

we extend the window and update:

```java
sum += nums[j];
```

This reduces complexity to:

```text
O(n²)
```

---

## Example Walkthrough

```text
nums = [1,100,1]
x = 1
```

### Start at index 0

```text
[1]
sum = 1
```

First digit = 1

Last digit = 1

Valid ✓

---

```text
[1,100]
sum = 101
```

First digit = 1

Last digit = 1

Valid ✓

---

```text
[1,100,1]
sum = 102
```

First digit = 1

Last digit = 2

Invalid ✗

---

### Start at index 1

```text
[100]
sum = 100
```

First digit = 1

Last digit = 0

Invalid ✗

---

```text
[100,1]
sum = 101
```

First digit = 1

Last digit = 1

Valid ✓

---

### Start at index 2

```text
[1]
sum = 1
```

Valid ✓

Answer:

```text
4
```

---

## Helper Function

To find the first digit:

```java
private int firstInteger(long n){
    while(n >= 10){
        n /= 10;
    }
    return (int)n;
}
```

Example:

```text
101 -> 10 -> 1
5678 -> 567 -> 56 -> 5
```

---

## Complexity Analysis

### Time Complexity

Outer loop:

```text
O(n)
```

Inner loop:

```text
O(n)
```

Running sum update:

```text
O(1)
```

First digit extraction:

```text
O(log(sum))
```

Maximum sum:

```text
1500 × 10^9 = 1.5 × 10^12
```

which has only about 13 digits.

Therefore:

```text
O(n²)
```

overall.

### Space Complexity

```text
O(1)
```

---

## Java Solution

```java
class Solution {
    public int countValidSubarrays(int[] nums, int x) {

        int result = 0;

        for (int i = 0; i < nums.length; i++) {

            long sum = 0;

            for (int j = i; j < nums.length; j++) {

                sum += nums[j];

                if (sum % 10 == x && firstInteger(sum) == x) {
                    result++;
                }
            }
        }

        return result;
    }

    private int firstInteger(long n) {

        while (n >= 10) {
            n /= 10;
        }

        return (int) n;
    }
}
```
