# Course Schedule

**Pattern:** Graph Traversal — DFS Cycle Detection (Directed Graph)

## Problem
Given `numCourses` and prerequisite pairs `[a, b]` (course `a` requires course
`b` first), determine if it's possible to finish all courses — i.e., does the
prerequisite graph contain a cycle?

LeetCode 207: https://leetcode.com/problems/course-schedule/

## Core insight — two sets, not one
Previous DFS problems (Max Area of Island, Closed Islands) used a single
`visited` marking: once visited, permanently excluded, never revisited.
**Cycle detection needs a second, separate kind of tracking**, because two
different situations look superficially similar but mean very different
things:

- **Diamond shape** (`A→B, A→C, B→D, C→D`): `D` gets visited twice — once via
  `B`, once via `C`. **Not a cycle** — `D`'s first DFS call had already fully
  *returned* (off the call stack) by the time the second path reaches it.
- **True cycle** (`A→B, B→C, C→A`): when `C`'s DFS tries to recurse into `A`,
  `A`'s own DFS call is **still in progress** — still on the call stack,
  waiting for its own recursive call (into `B`) to return. That's the
  signature of a cycle: revisiting a node that is a currently-active ancestor
  in the current DFS path, not one that's already finished.

**The fix:** maintain two sets:
- `visited` — "fully finished exploring this node, confirmed safe" (permanent,
  like island-marking)
- `inProgress` (a.k.a. "on current path" / recursion stack tracking) — "this
  node's DFS call has started but not yet returned." Added at the **start**
  of a node's `dfs` call, removed right **before** that call returns.

If a neighbor is found in `inProgress` → cycle detected. If found in
`visited` → already safe, skip (not a cycle, just paths converging). If in
neither → recurse.

## Algorithm
```
dfs(node):
    inProgress.add(node)
    for each neighbor n of node:
        if n in inProgress: return true     // cycle found
        if n in visited: continue           // already safe, skip
        if dfs(n) == true: return true      // cycle found deeper
    inProgress.remove(node)
    visited.add(node)
    return false                            // no cycle through this node
```
Convention chosen: `dfs` returns `true` when a cycle **is** found (simpler
than the reverse — avoids an extra negation when combining results in the
outer loop).

**Outer loop** (handles disconnected components, same pattern as
[[is-graph-bipartite-bfs-coloring]]): loop over all courses `0` to
`numCourses-1`, skip if already in `visited`, call `dfs`. If any call returns
`true`, the whole graph has a cycle → `canFinish` returns `false` immediately.
If the loop completes with no cycles anywhere, return `true`.

**Edge case caught during coding:** a node with no outgoing edges at all
(`map.get(source) == null`) still needs its `inProgress`→`visited` cleanup
performed before returning `false` — otherwise it would incorrectly remain
"in progress" forever.

## Edge direction — doesn't matter for cycle detection specifically
Given `pr[i] = [a, b]` (course `a` needs prerequisite `b`), the adjacency list
could point either `a→b` or `b→a` — both work correctly for *this* problem,
since a cycle exists in the dependency structure regardless of which way the
edges are drawn (reversing every edge of a cyclic graph keeps it cyclic).
Direction matters much more for other graph problems (e.g., topological
sort), so it's worth being deliberate about which direction is chosen and why,
even when — as here — either choice is technically valid.

## Complexity
- **Time: O(V + E)** — same single-traversal reasoning as standard BFS/DFS:
  once a node is in `visited`, it's never explored again. The added
  `inProgress` check is just one more O(1) Set lookup per edge examined during
  the same traversal — no extra pass over the data, so it doesn't change the
  complexity class, only adds a constant factor.
- **Space: O(V + E)** — adjacency list is O(E), `visited` and `inProgress`
  are each O(V) in the worst case (two Sets instead of one doesn't change the
  complexity *class*, just a constant-factor increase), and the recursion
  stack can also reach O(V) depth in the worst case (a long dependency chain).

## Solution

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] pr) {
        Map<Integer,List<Integer>> map = new HashMap<>();

        for (int i = 0; i < pr.length; i++) {
            int[] arr = pr[i];
            int key = arr[1];   // prerequisite
            int value = arr[0]; // course that needs it
            if (map.containsKey(key)) {
                map.get(key).add(value);
            } else {
                List<Integer> list = new ArrayList<>();
                list.add(value);
                map.put(key, list);
            }
        }

        Set<Integer> visited = new HashSet<>();
        Set<Integer> inProgress = new HashSet<>();
        for (int i = 0; i < numCourses; i++) {
            if (!visited.contains(i)) {
                if (dfs(i, map, visited, inProgress)) return false;
            }
        }
        return true;
    }

    private boolean dfs(int source, Map<Integer,List<Integer>> map, Set<Integer> visited, Set<Integer> inProgress) {
        inProgress.add(source);
        List<Integer> values = map.get(source);
        if (values == null) {
            inProgress.remove(source);
            visited.add(source);
            return false;
        }
        for (int i : values) {
            if (inProgress.contains(i)) return true;
            if (visited.contains(i)) continue;
            if (dfs(i, map, visited, inProgress)) return true;
        }
        inProgress.remove(source);
        visited.add(source);
        return false;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| First attempt: `int[] arr = pr[i][0];` — tried to assign a single `int` (`pr[i][0]`) to an `int[]` variable | `pr[i]` is already the full row (`int[]`); `pr[i][0]` is a single `int` value. Should be `int[] arr = pr[i];` |
| Initially proposed using two Queues to detect cycles, trying to force a BFS-shaped solution | Cycle detection via "is this node currently an ancestor on my active path" is naturally a recursion + two-Set (visited/inProgress) problem — the call stack itself tracks "in progress" implicitly, which a queue-based approach doesn't provide |

**Process note:** pseudocode was fully written and verified (including explicit
reasoning about the diamond-vs-cycle distinction) before any Java was
written. Only one real syntax bug appeared in the first full code attempt —
continuing the trend of meaningfully fewer bugs since committing to
plan-first, trace-before-pasting.
