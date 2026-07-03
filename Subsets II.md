# Subsets II — Notes

> **Problem:** Given an integer array that may contain duplicates, return all possible subsets without duplicates.
> LC 90 — leetcode.com/problems/subsets-with-duplicates

---

## The pattern — subsets with duplicate pruning

Same as Subsets I (record every state, loop from start), but with two additions:
1. **Sort first** — groups duplicates adjacent to each other
2. **Pruning condition** — skip a value if the identical value at the previous index was already tried at this same level

---

## Solution

```java
class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);                        // sort first — groups duplicates
        List<List<Integer>> result = new ArrayList<>();
        function(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private void function(int[] nums, int start, List<Integer> temp, List<List<Integer>> result) {
        result.add(new ArrayList<>(temp));        // record every state immediately

        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i-1]) continue;  // skip duplicates

            temp.add(nums[i]);                    // CHOOSE
            function(nums, i+1, temp, result);    // EXPLORE
            temp.remove(temp.size()-1);           // UNCHOOSE
        }
    }
}
```

---

## Why brute force (contains check) breaks for duplicates

```java
// ❌ brute force — breaks for [4,4,4,1,4]
private void function(int[] nums, int k, List<Integer> temp, List<List<Integer>> result) {
    if (k == nums.length) {
        if (!result.contains(temp)) result.add(new ArrayList<>(temp));
        return;
    }
    temp.add(nums[k]);
    function(nums, k+1, temp, result);
    temp.remove(temp.size()-1);
    function(nums, k+1, temp, result);
}
```

**Input: `[4,4,4,1,4]`**

Without sorting, subsets are built in input order — `1` is in the middle, so you get `[4,4,4,1,4]` instead of `[1,4,4,4,4]`. The `contains` check compares `[4,4,4,1,4]` vs `[1,4,4,4,4]` — different lists even though same elements — so duplicates slip through.

**Two root causes:**
1. Input not sorted → subsets generated in wrong order
2. `contains` compares order-sensitive lists → misses element-equivalent duplicates

**Cannot be patched** — would need to sort every `temp` before contains check, making it even slower. Sort + prune is the correct structural fix.

---

## The pruning condition — explained

```java
if (i > start && nums[i] == nums[i-1]) continue;
```

**`nums[i] == nums[i-1]`** — current value is same as previous. After sorting, duplicates are adjacent, so this catches them.

**`i > start`** — only skip duplicates tried **at this exact loop level**, not duplicates from deeper levels.

---

## Why `i > start` not `i > 0`

`i > 0` would be too aggressive — it would skip valid subsets:

```
function(start=0, temp=[1])  ← already picked 1
  i=1: pick nums[1]=4 → [1,4] ✅
  i=2: nums[2]==nums[1] && i>0=true → SKIP ❌ misses [1,4,4]
```

`[1,4]` and `[1,4,4]` are genuinely different subsets. `nums[1]=4` was picked in a **different recursive call** at a deeper level — not at this same loop level. We should not skip here.

`i > start` means: "skip only if same value as something already tried at this exact loop level":

```
function(start=1, temp=[])
  i=1: pick nums[1]=4 → generates [4],[4,4],[4,4,4],[4,4,4,4] ✅
  i=2: nums[2]==nums[1] && i>start=1 → SKIP ✅ same level, same value, already explored
  i=3: SKIP ✅
  i=4: SKIP ✅
```

---

## Visual — what gets skipped vs allowed for `[1,4,4,4,4]`

```
function(start=1, temp=[])  ← choosing first element from the 4s

index:  1   2   3   4
value:  4   4   4   4
        ↓   ↓   ↓   ↓
      ALLOW SKIP SKIP SKIP

i=1: i==start → always allow first pick at this level
i=2: nums[2]==nums[1] && i>start → SKIP — would duplicate everything i=1 generated
i=3: SKIP
i=4: SKIP
```

---

## Trace for `[1,1,2]`

```
sort → [1,1,2]

function(start=0, temp=[])
  record []
  i=0: pick 1 → temp=[1]
    function(start=1, temp=[1])
      record [1]
      i=1: pick 1 → temp=[1,1]
        function(start=2, temp=[1,1])
          record [1,1]
          i=2: pick 2 → temp=[1,1,2]
            record [1,1,2] ✅
          unpick 2
        return
      unpick 1
      i=2: pick 2 → temp=[1,2]
        record [1,2] ✅
      unpick 2
    return
  unpick 1
  i=1: nums[1]==nums[0] && i>start=0 → SKIP
  i=2: pick 2 → temp=[2]
    record [2] ✅
  unpick 2

result = [[], [1], [1,1], [1,1,2], [1,2], [2]] ✅
```

---

## Subsets I vs Subsets II

| | Subsets I | Subsets II |
|---|---|---|
| Input | no duplicates | may have duplicates |
| Sort needed | no | yes — groups duplicates |
| Pruning condition | none | `i>start && nums[i]==nums[i-1]` |
| Record when | every call | every call |
| Base case | implicit — loop ends | implicit — loop ends |

---

## Subsets II vs Permutations II — pruning comparison

| | Subsets II | Permutations II |
|---|---|---|
| Pruning condition | `i>start && nums[i]==nums[i-1]` | `i>0 && nums[i]==nums[i-1] && !used[i-1]` |
| Why simpler | loop already starts from `start` — no need for `used[]` | loop starts from 0 every time — needs `used[]` to track what's placed |
| `i > start` vs `i > 0` | `i > start` — skip only at same level | `i > 0` with `!used[i-1]` to distinguish levels |

---

## One line to remember

> Sort first. Record every state. Skip `nums[i]` if it equals `nums[i-1]` AND `i > start` — same value already explored at this level, skipping prevents duplicate subsets.
