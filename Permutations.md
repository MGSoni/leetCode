# 46. Permutations

## Problem

Given an array `nums` of distinct integers, return all the possible permutations. You can return the answer in any order.

### Example 1

Input:

```text
nums = [1,2,3]
```

Output:

```text
[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

### Example 2

Input:

```text
nums = [0,1]
```

Output:

```text
[[0,1],[1,0]]
```

### Example 3

Input:

```text
nums = [1]
```

Output:

```text
[[1]]
```

### Constraints

```text
1 <= nums.length <= 6
-10 <= nums[i] <= 10
All integers in nums are unique.
```

---

## Pattern Recognition

This is a classic **Backtracking** problem.

Keywords that hint toward backtracking:

* Generate all permutations.
* Need every possible arrangement.
* Elements cannot be reused within the same permutation.
* Small constraint:

```text
nums.length <= 6
```

Since:

```text
6! = 720
```

generating all permutations is feasible.

---

## Key Idea

Build the permutation one element at a time.

For every position:

* Try every unused number.
* Add it to the current permutation.
* Recurse for the next position.
* Backtrack by removing it before trying the next choice.

---

## State Variables

### current

Stores the permutation currently being built.

Example:

```text
[1, 3]
```

means:

```text
We have already chosen 1 and 3.
```

---

### used

Tracks whether a number has already been selected.

Example:

```text
nums = [1,2,3]

used = [true,false,true]
```

means:

```text
1 and 3 are already present in current.
2 is still available.
```

---

## Backtracking Flow

Suppose:

```text
nums = [1,2,3]
```

Initially:

```text
current = []
used = [F,F,F]
```

Choose:

```text
1
```

```text
current = [1]
used = [T,F,F]
```

Choose:

```text
2
```

```text
current = [1,2]
used = [T,T,F]
```

Choose:

```text
3
```

```text
current = [1,2,3]
used = [T,T,T]
```

Length equals:

```text
nums.length
```

Store the permutation.

---

## Why Backtracking?

After exploring:

```text
[1,2,3]
```

we must explore:

```text
[1,3,2]
```

Therefore we undo the last choice:

```java
current.remove(current.size() - 1);
used[i] = false;
```

This restores the state before the recursive call.

---

## Recursion Tree

```text
                    []
           /         |         \
          1          2          3
        /   \      /   \      /   \
       2     3    1     3    1     2
      /       \  /       \  /       \
     3         2 3        1 2        1
```

Generated permutations:

```text
[1,2,3]
[1,3,2]
[2,1,3]
[2,3,1]
[3,1,2]
[3,2,1]
```

---

## Base Case

When:

```java
current.size() == nums.length
```

a complete permutation has been formed.

Store a copy:

```java
result.add(new ArrayList<>(current));
```

We must create a new ArrayList because current will continue changing during backtracking.

---

## Important Mistake to Avoid

Wrong:

```java
result.add(current);
```

This stores the same reference repeatedly.

As backtracking modifies current, all entries in result would change.

Correct:

```java
result.add(new ArrayList<>(current));
```

which stores an independent copy.

---

## Complexity Analysis

### Time Complexity

There are:

```text
n!
```

permutations.

Each permutation requires:

```text
O(n)
```

to copy into the result.

Therefore:

```text
O(n × n!)
```

---

### Space Complexity

Recursion depth:

```text
O(n)
```

Used array:

```text
O(n)
```

Ignoring output storage:

```text
O(n)
```

Including output:

```text
O(n × n!)
```

---

## Java Solution

```java
class Solution {

    public List<List<Integer>> permute(int[] nums) {

        List<List<Integer>> result = new ArrayList<>();

        function(
            nums,
            new ArrayList<Integer>(),
            result,
            new boolean[nums.length]
        );

        return result;
    }

    private void function(
        int[] nums,
        List<Integer> current,
        List<List<Integer>> result,
        boolean[] used
    ) {

        if (current.size() == nums.length) {
            result.add(new ArrayList<>(current));
            return;
        }

        for (int i = 0; i < nums.length; i++) {

            if (used[i]) {
                continue;
            }

            current.add(nums[i]);
            used[i] = true;

            function(nums, current, result, used);

            current.remove(current.size() - 1);
            used[i] = false;
        }
    }
}
```
SOLUTION WITHOUT EXTRA SPACE

class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        function(nums, result, 0);
        return result;
    }

    private void function (int[] nums, List<List<Integer>> result, int k){
        if(k == nums.length){
            List<Integer> list = new ArrayList<>();
             for(int i=0;i<nums.length;i++){
                 list.add(nums[i]);
             }
             result.add(list);
             return;
        }else{
            int temp = 0;
            for(int i=k; i<nums.length ; i++){
                temp = nums[k];
                nums[k] =nums[i];
                nums[i] = temp;
                function(nums, result, k+1);
                temp = nums[k];
                nums[k] =nums[i];
                nums[i] = temp;
            }
        }

        
    }
}
