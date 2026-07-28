# Combinations — Notes

> **Problem:** Given `n` and `k`, find all ways to pick `k` numbers from `[1..n]` where order doesn't matter.
> LC 77 — leetcode.com/problems/combinations

---

## The pattern

Combinations = backtracking where you **never go backwards**.

At each step, pick a number, recurse on everything after it, unpick.
Passing `i+1` into the recursive call ensures you only ever move forward — no duplicates like `[2,1]` after `[1,2]`.

---

## The solution

```java
void function(int n, int k, int start, List<Integer> combination, List<List<Integer>> result) {
    if (combination.size() == k) {
        result.add(new ArrayList<>(combination));   // base case — k numbers picked
        return;
    }

    for (int i = start; i <= n; i++) {
        combination.add(i);                         // CHOOSE
        function(n, k, i+1, combination, result);   // EXPLORE — pass i+1, not start+1
        combination.remove(combination.size()-1);   // UNCHOOSE
    }
}
```

---

## The one bug to never make again

```java
function(n, k, value+1, combination, result);  // ❌ wrong
function(n, k, i+1, combination, result);      // ✅ correct
```

These look almost identical. The difference is everything.

---

## Why `i+1` and not `start+1`

`start` and `i` are two different things playing two different roles:

| | What it is | What it answers |
|---|---|---|
| `start` | where this call begins | "what did my parent tell me?" |
| `i` | current loop position | "what did I just pick?" |

The next call needs to know what **you just picked** (`i`), not where **you started** (`start`). Those are only the same on the first iteration — after that they diverge.

**Concrete example:** `start=2`, loop is at `i=3`, you just picked 3.
- Pass `start+1 = 3` → next call can pick 3 again → `[3,3]` ❌
- Pass `i+1 = 4` → next call picks only after 3 → correct ✅

---

## The promise chain mental model

Each call **receives** a promise from its parent and **makes** a new promise to its child:

```
grandparent picked 1 → told parent "start from 2"
parent picked 3      → tells child "start from 4"
child picks 4        → tells grandchild "start from 5"
```

- `start` = the promise you **received** (not yours to pass on)
- `i+1` = the promise you **make** (based on your own pick)

Passing `start+1` means passing someone else's promise down. Passing `i+1` means passing your own.

---

## How combinations differ from permutations and binary strings

| Problem | Parameter role | Pass into recursive call |
|---|---|---|
| Binary strings | `k` = current slot position | always `k+1` (slot advances by 1 regardless of choice) |
| Permutations | `used[]` = which numbers are taken | loop from `0`, skip used ones |
| Combinations | `start` = minimum allowed pick | `i+1` (depends on what you just picked) |

---

## The cheat code — ask this before every recursive call

> **"What did I just do, and how does that change what the next call is allowed to do?"**

Fill in the blank:

```
I just _____________, so the next call should start from _____________.
```

Applied here:
```
I just picked number i, so the next call should start from i+1
(can't pick i or anything before it again).
→ pass i+1 ✅
```

---

## Trace for `n=4, k=2`

```
start=1
  i=1: pick 1 → [1]
    i=2: pick 2 → [1,2] → record ✅
    i=3: pick 3 → [1,3] → record ✅
    i=4: pick 4 → [1,4] → record ✅
  i=2: pick 2 → [2]
    i=3: pick 3 → [2,3] → record ✅
    i=4: pick 4 → [2,4] → record ✅
  i=3: pick 3 → [3]
    i=4: pick 4 → [3,4] → record ✅
  i=4: pick 4 → [4]
    loop ends, nothing to pick → no record

result = [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
```
