# Course Schedule II

**Pattern:** Graph Traversal — DFS Topological Sort

## Problem
Given `numCourses` and prerequisite pairs `[a, b]` (course `a` requires course
`b` first), return a valid order to take all courses, or an empty array if
impossible (cycle exists).

LeetCode 210: https://leetcode.com/problems/course-schedule-ii/
(Direct follow-up to [[course-schedule-cycle-detection-dfs]] — same cycle
detection requirement, plus now needing to produce an actual ordering.)

## Core insight — finish order gives you the answer directly
If `b` is a prerequisite of `a`, then in a DFS where `a`'s traversal recurses
into `b`, `b`'s `dfs` call **finishes and returns before** `a`'s does (since
`a`'s call can't return until the nested call into `b` completes). So if you
record each node **at the moment its `dfs` call finishes** (not when it
starts), the resulting list already has prerequisites appearing before the
courses that need them — **no reversal needed**, given the edge direction
chosen below.

## Critical direction difference from Course Schedule (LC 207)
In LC 207, the adjacency list was built `b → a` (prerequisite points to what
it unlocks). **Here, the direction must be flipped: `a → b`** (course points
to its prerequisite), so that `dfs(a)` recurses into `dfs(b)` — ensuring `b`
(the prerequisite) finishes first and lands earlier in the finish-order list.
Verified by hand-tracing both a 1-edge and 2-edge chain
(`[[1,0]]` → order `[0,1]`; `[[1,0],[2,1]]` → order `[0,1,2]`) before trusting
the direction. **Don't assume the same adjacency direction from a related
problem carries over — re-derive it for the new problem's requirements.**

## Algorithm
Same two-set cycle detection as Course Schedule (`visited` / `inProgress`),
plus an `orderList` accumulator appended to at every "node is fully finished"
point:

```
dfs(source, map, orderList, visited, inProgress):
    neighbors = map.get(source)
    if neighbors == null:
        visited.add(source)
        orderList.add(source)
        return false
    inProgress.add(source)
    for each n in neighbors:
        if n in inProgress: return true
        if n in visited: continue
        if dfs(n, ...) == true: return true
    inProgress.remove(source)
    visited.add(source)
    orderList.add(source)
    return false
```

**Note on `inProgress.add` placement:** this version adds to `inProgress`
*after* the null check (not unconditionally at the very top, as in the
original Course Schedule pseudocode). This is still safe — a node with zero
outgoing edges can never be part of a cycle (a cycle requires a path that
loops back; a dead-end node has nowhere to loop to) — but it's a slightly
fragile shortcut. The more robust default is marking `inProgress`
unconditionally at the top of every call, even when a specific edge case
happens to be safe either way.

**Outer loop:** same disconnected-components pattern — loop `0` to
`numCourses-1`, skip if visited, call `dfs`. If any call returns `true`
(cycle found), return `new int[0]` immediately. Otherwise, convert
`orderList` (a `List<Integer>`) to `int[]` with a manual loop and return it.

## Complexity
- **Time: O(V + E)** — same reasoning as every prior graph traversal: each
  node explored once, each edge examined once across the whole run.
- **Space: O(V + E)** — adjacency list O(E); `visited`, `inProgress`, and the
  new `orderList` are each O(V) — `orderList` holds at most one entry per
  course, so it doesn't change the complexity class, same order as the
  existing sets.

## Solution

```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        Map<Integer,List<Integer>> map = new HashMap<>();

        for (int i = 0; i < prerequisites.length; i++) {
            int[] arr = prerequisites[i];
            int key = arr[0];   // course
            int value = arr[1]; // prerequisite
            if (map.containsKey(key)) {
                map.get(key).add(value);
            } else {
                List<Integer> list = new ArrayList<>();
                list.add(value);
                map.put(key, list);
            }
        }

        List<Integer> orderList = new ArrayList<>();
        Set<Integer> visited = new HashSet<>();
        Set<Integer> inProgress = new HashSet<>();

        for (int i = 0; i < numCourses; i++) {
            if (!visited.contains(i)) {
                if (dfs(i, map, orderList, visited, inProgress)) {
                    return new int[0];
                }
            }
        }

        int[] result = new int[orderList.size()];
        for (int i = 0; i < orderList.size(); i++) {
            result[i] = orderList.get(i);
        }
        return result;
    }

    private boolean dfs(int source, Map<Integer,List<Integer>> map, List<Integer> orderList, Set<Integer> visited, Set<Integer> inProgress) {
        List<Integer> list = map.get(source);
        if (list == null) {
            visited.add(source);
            orderList.add(source);
            return false;
        }
        inProgress.add(source);
        for (int i : list) {
            if (inProgress.contains(i)) return true;
            if (visited.contains(i)) continue;
            if (dfs(i, map, orderList, visited, inProgress)) return true;
        }
        inProgress.remove(source);
        visited.add(source);
        orderList.add(source);
        return false;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Initially assumed the same adjacency-list direction as Course Schedule (`b→a`) would work here too | Re-derived from scratch via hand-tracing — topological sort via finish-order needs the opposite direction (`a→b`) so prerequisites finish (and get recorded) before the courses depending on them |
| First pseudocode draft checked `neighbors == null` *before* marking `inProgress`, and skipped appending to `orderList`/`visited` on the null path entirely | A node must get its full "finished" bookkeeping (visited + orderList, and in the general case inProgress cleanup) regardless of *which* return path finishes it — the null case is just "zero loop iterations," not an exemption from bookkeeping |
| Got a runtime `UnsupportedOperationException` pointing at a `.add()` call on an immutable list — but the exact code pasted didn't contain any `List.of(...)` or other immutable-list-creating call | Turned out to be a stale/mismatched version run on the judge, not the pasted code. Worth double-checking that the code actually submitted matches what's being debugged when a stack trace doesn't line up with the pasted source |

**Process note:** topological sort direction was verified by hand-tracing two
concrete examples (1-edge and 2-edge chains) before trusting it, rather than
assuming the Course Schedule direction would carry over unchanged — a good
habit to keep for any problem that reuses a structure from a related but
distinct problem.
