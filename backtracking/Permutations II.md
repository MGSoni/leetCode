# Permutations II — Notes

> **Problem:** Given a collection of numbers that might contain duplicates, return all unique permutations.
> LC 47 — leetcode.com/problems/permutations-ii

---

## Two approaches

### Approach 1 — Brute force (contains check)
### Approach 2 — Optimized (sort + prune) ← preferred in interviews

---

## Approach 1 — Brute force using contains check

Generate all permutations normally (swap-based), check if result already contains the permutation before adding.

```java
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        function(nums, 0, result);
        return result;
    }

    private void function(int[] nums, int k, List<List<Integer>> result) {
        if (k == nums.length) {
            List<Integer> temp = new ArrayList<>();
            for (int i : nums) temp.add(i);
            if (!result.contains(temp)) result.add(temp);  // check before adding
            return;
        }
        for (int i = k; i < nums.length; i++) {
            int tmp = nums[i]; nums[i] = nums[k]; nums[k] = tmp;
            function(nums, k+1, result);
            tmp = nums[i]; nums[i] = nums[k]; nums[k] = tmp;
        }
    }
}
```

**Works correctly but inefficient:**
- `result.contains(temp)` scans the entire result list at every base case
- O(n! × n) in worst case — checks n! permutations, each check costs O(n)
- Fine for small inputs, times out on large inputs

---

## Approach 2 — Optimized using sort + prune

## The pattern — backtracking with duplicate pruning

Same as Permutations I (used[] array, loop from 0), but with two additions:
1. **Sort first** — groups duplicates adjacent to each other
2. **Pruning condition** — skip a value if an identical value at the previous index already explored and backtracked

---

## Solution

```java
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);                        // step 1 — sort to group duplicates
        boolean[] used = new boolean[nums.length];
        List<List<Integer>> result = new ArrayList<>();
        function(nums, result, used, new ArrayList<>());
        return result;
    }

    private void function(int[] nums, List<List<Integer>> result,
                          boolean[] used, List<Integer> temp) {
        if (temp.size() == nums.length) {
            result.add(new ArrayList<>(temp));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;                // skip already placed numbers

            // pruning — skip duplicate value if previous copy already backtracked
            if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;

            used[i] = true;
            temp.add(nums[i]);                    // CHOOSE
            function(nums, result, used, temp);   // EXPLORE
            temp.remove(temp.size()-1);           // UNCHOOSE
            used[i] = false;
        }
    }
}
```

---

## My mistake — unused parameter

```java
// ❌ passing i+1 as k but k is never used inside the function
function(nums, i+1, result, used, temp);

// ✅ permutations always loop from 0 — used[] handles skipping, not a start index
function(nums, result, used, temp);
```

Passing `i+1` is combinations logic — where you move forward to avoid going backwards. In permutations, you always try all indices from `0` and rely on `used[]` to skip placed elements.

---

## The pruning condition — explained

```java
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```

Three parts:

**`i > 0`** — safety check, ensures `nums[i-1]` exists.

**`nums[i] == nums[i-1]`** — current value is a duplicate of the previous one.

**`!used[i-1]`** — the previous copy is NOT currently in the path (it already explored and backtracked).

If all three are true → skip. Picking this value would generate the exact same permutations the previous copy already generated.

---

## Two scenarios — when to skip, when not to

**Allow — `used[i-1] = true` (previous copy IS in path):**
```
Path = [1, ...]   ← first 1 placed, used[0]=true
i=1: nums[1]=1, nums[0]=1, !used[0] = false → DON'T skip
Picking second 1 at a different position → genuinely different permutation ✅
```

**Skip — `used[i-1] = false` (previous copy already backtracked):**
```
Path = []   ← filling position 0
i=0: pick first 1 → explore → [1,1,2],[1,2,1] generated → backtrack → used[0]=false
i=1: nums[1]=1, nums[0]=1, !used[0] = true → SKIP
Would generate [1,1,2],[1,2,1] again — duplicates ❌
```

---

## Trace for `[1,1,2]`

```
sort → [1,1,2]

filling position 0:
  i=0: pick 1 (first) → explore
    filling position 1:
      i=1: pick 1 (second) → explore
        filling position 2: pick 2 → record [1,1,2] ✅
      i=2: pick 2 → explore
        filling position 2: pick 1 (second) → record [1,2,1] ✅
  i=1: nums[1]==nums[0] && !used[0] → SKIP (first 1 already backtracked)
  i=2: pick 2 → explore
    filling position 1:
      i=0: pick 1 (first) → explore
        filling position 2: i=1 pick 1 (second) → record [2,1,1] ✅
      i=1: nums[1]==nums[0] && !used[0] → SKIP

result = [[1,1,2], [1,2,1], [2,1,1]] ✅
```

---

## Permutations I vs Permutations II

| | Permutations I | Permutations II |
|---|---|---|
| Input | no duplicates | may have duplicates |
| Sort needed | no | yes — groups duplicates |
| Pruning condition | none | `i>0 && nums[i]==nums[i-1] && !used[i-1]` |
| Loop starts from | 0 always | 0 always |
| Duplicate prevention | not needed | sort + prune before recursing |

---

## Approach 1 vs Approach 2

| | Brute force (contains) | Optimized (sort + prune) |
|---|---|---|
| Correctness | ✅ correct | ✅ correct |
| Time complexity | O(n! × n) | O(n! × n) worst case but prunes early |
| Duplicate handling | after generating — check result list | before generating — skip at source |
| Interview preferred | ❌ | ✅ |
| When to use | explaining thought process first | final answer |

---

## One line to remember

> Sort first to group duplicates. Skip a value if the identical value before it already backtracked — it already explored all permutations for that value at this position.
