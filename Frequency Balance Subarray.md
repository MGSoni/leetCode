# 3960. Frequency Balance Subarray

## Problem

You are given an integer array nums.

Define a frequency balance subarray as follows:

* If the subarray contains only one distinct value, it is frequency balanced.
* Otherwise, there must exist a positive integer f such that every distinct value in the subarray occurs either f or 2 * f times, and both frequencies occur among the distinct values.

Return an integer denoting the length of the longest frequency balance subarray.

### Example 1

Input:

```text
nums = [1,2,2,1,2,3,3,3]
```

Output:

```text
5
```

Explanation:

The longest frequency balance subarray is:

```text
[2,1,2,3,3]
```

The elements that appear most frequently are 2 and 3, both appearing twice.

The remaining element 1 appears once, meeting the requirements.

### Example 2

Input:

```text
nums = [5,5,5,5]
```

Output:

```text
4
```

Explanation:

The longest frequency balance subarray is:

```text
[5,5,5,5]
```

The element appears only once as a distinct value.

### Example 3

Input:

```text
nums = [1,2,3,4]
```

Output:

```text
1
```

### Constraints

```text
1 <= nums.length <= 1000
1 <= nums[i] <= 10^9
```

---

## Pattern Recognition

* Not Sliding Window (condition is not monotonic).
* Not Prefix Sum.
* Since n ≤ 1000, O(n²) brute force is acceptable.

---

## Mistake I Made

I interpreted:

```text
there exists a positive integer f
```

as if f was an element in the array.

I also missed:

```text
both frequencies occur among the distinct values
```

and thought frequencies could be either f or 2f, without requiring both to be present.

Actual condition:

```text
All frequencies ∈ {f, 2f}
AND
Both f and 2f must appear.
```

---

## Approach

For every starting index i:

* Extend j from i to n-1.
* Maintain frequency of each value.
* Maintain frequency of frequencies.
* Check whether the current window is balanced.

### Data Structures

```java
freqMap
value -> frequency
```

```java
freqOfFreq
frequency -> count of values having that frequency
```

Example:

```text
[2,1,2,3,3]

freqMap:
{
    2:2,
    1:1,
    3:2
}

freqOfFreq:
{
    1:1,
    2:2
}
```

---

## Important Bug

Wrong:

```java
if(freqOfFreq.size() == 1)
```

Correct:

```java
if(freqMap.size() == 1)
```

Reason:

```text
freqMap.size()
= number of distinct values

freqOfFreq.size()
= number of distinct frequencies
```

Example:

```text
[1,2]

freqMap = {1:1, 2:1}
freqMap.size() = 2

freqOfFreq = {1:2}
freqOfFreq.size() = 1
```

This subarray is NOT balanced, so checking `freqOfFreq.size()==1` is incorrect.

---

## Complexity

```text
Time  : O(n²)
Space : O(n)
```

---

## Solution

```java
import java.util.*;

class Solution {
    public int getLength(int[] nums) {
        int n = nums.length;
        int best = 1;

        for(int i = 0; i < n; i++) {

            Map<Integer, Integer> freqMap = new HashMap<>();
            Map<Integer, Integer> freqOfFreq = new HashMap<>();

            for(int j = i; j < n; j++) {

                int x = nums[j];

                int oldFreq = freqMap.getOrDefault(x, 0);
                int updatedFreq = oldFreq + 1;

                if(oldFreq > 0) {
                    int freqOfOldFreq = freqOfFreq.get(oldFreq);

                    if(freqOfOldFreq == 1) {
                        freqOfFreq.remove(oldFreq);
                    } else {
                        freqOfFreq.put(oldFreq, freqOfOldFreq - 1);
                    }
                }

                freqMap.put(x, updatedFreq);

                freqOfFreq.put(
                    updatedFreq,
                    freqOfFreq.getOrDefault(updatedFreq, 0) + 1
                );

                int length = j - i + 1;

                if(freqMap.size() == 1) {
                    best = Math.max(best, length);
                }
                else if(freqOfFreq.size() == 2) {

                    Iterator<Integer> it = freqOfFreq.keySet().iterator();

                    int a = it.next();
                    int b = it.next();

                    if(a > b) {
                        int t = a;
                        a = b;
                        b = t;
                    }

                    if(b == 2 * a) {
                        best = Math.max(best, length);
                    }
                }
            }
        }

        return best;
    }
}
```
