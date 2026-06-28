# Subsets — Notes

> **Problem:** Given an array, return all possible subsets including the empty set.
> LC 78 — leetcode.com/problems/subsets

---

## The pattern

Subsets = backtracking where **every state along the way is a valid answer**.

Record at the top of every call, before the loop runs. No size limit, no explicit base case needed.

---

## My mistake

```java
private void function(int[] nums, int index, List<Integer> temp, List<List<Integer>> result){
    if(index == nums.length){
        result.add(new ArrayList<>(temp));  // ❌ recording at base case
        return;
    }

    result.add(new ArrayList<>(temp));      // ❌ also recording at top of call
    for(int i=index; i<nums.length; i++){
        temp.add(nums[i]);
        function(nums, i+1, temp, result);
        temp.remove(temp.size()-1);
    }
}
```

**Two bugs:**

**Bug 1 — Recording in two places → duplicates**

Every subset got recorded twice:
- once at the top of each call
- once again when it hit the base case

**Bug 2 — Unnecessary explicit base case**

The loop condition `i < nums.length` becomes false naturally when `start == nums.length` — the function returns on its own without needing an explicit check.

---

## Correct solution

```java
private void function(int[] nums, int start, List<Integer> temp, List<List<Integer>> result) {
    result.add(new ArrayList<>(temp));        // record current subset — always, immediately

    for (int i = start; i < nums.length; i++) {
        temp.add(nums[i]);                    // CHOOSE
        function(nums, i+1, temp, result);    // EXPLORE
        temp.remove(temp.size()-1);           // UNCHOOSE
    }
}
```

---

## Why no explicit base case is needed

Every recursive call passes `i+1` as the new `start`. So `start` grows by at least 1 every call. Eventually `start == nums.length` and the loop condition `i < nums.length` is false from the first iteration — loop never runs, function records and returns naturally.

```
function(start=2, temp=[1,2])   // nums.length = 2
  record [1,2]
  loop: i=2, 2 < 2 → false → loop never runs
  return ← stops here naturally, no crash
```

**Rule of thumb:**
> If the loop condition becomes false before anything dangerous happens, the loop is your base case — no explicit check needed.
> If the recursion can reach a null or invalid state, add an explicit base case to catch it.

---

## Subsets vs Combinations — the one difference

| | When to record | Base case |
|---|---|---|
| Combinations | only when `size == k` | explicit — `if size == k` |
| Subsets | at every call, immediately | implicit — loop condition `i < n` |

Same skeleton. Same choose/explore/unchoose. Only the recording moment changes.

---

## Trace for `[1,2,3]`

```
function(start=0, temp=[])
  record []
  i=0: pick 1 → [1]
    function(start=1, temp=[1])
      record [1]
      i=1: pick 2 → [1,2]
        function(start=2, temp=[1,2])
          record [1,2]
          i=2: pick 3 → [1,2,3]
            function(start=3, temp=[1,2,3])
              record [1,2,3]
              loop: 3<3 false → return
          unpick 3
        return
      unpick 2
      i=2: pick 3 → [1,3]
        function(start=3, temp=[1,3])
          record [1,3]
          loop: 3<3 false → return
      unpick 3
    return
  unpick 1
  i=1: pick 2 → [2]
    function(start=2, temp=[2])
      record [2]
      i=2: pick 3 → [2,3]
        function(start=3, temp=[2,3])
          record [2,3]
          loop: 3<3 false → return
      unpick 3
    return
  unpick 2
  i=2: pick 3 → [3]
    function(start=3, temp=[3])
      record [3]
      loop: 3<3 false → return
  unpick 3

result = [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]  ✅
```
