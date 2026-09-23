# Is Graph Bipartite? (2-Coloring)

**Pattern:** Graph Traversal — BFS + 2-Coloring, Disconnected Components

## Problem
Given an undirected graph, determine if its nodes can be split into two groups
such that every edge connects a node in one group to a node in the other (no
edge has both endpoints in the same group). Graph may be disconnected.

LeetCode 785: https://leetcode.com/problems/is-graph-bipartite/description/
(Also seen as a Scaler variant taking `A` = node count and `B` = edge list.)

## Core insight — coloring is forced, not chosen
- If node `u` is colored (say) Red, then **every** neighbor of `u` is *forced*
  to be Blue — there's no choice once `u`'s color is fixed.
- Propagating this outward: if two different paths through the graph reach the
  same node `w` with **conflicting** color demands (one path says Red, another
  says Blue), the graph **cannot** be bipartite. A single such conflict is
  enough to return `false` immediately.
- This reduces the whole problem to: **BFS while trying to 2-color the graph,
  and detect if any conflict ever arises.**

## Adapting standard BFS for coloring
- Previous problems used a `Set<Integer>` for visited-tracking (yes/no). Here,
  you need more than yes/no — for each node, "is it colored, and if so, which
  color?" A `Map<Integer, Integer>` (node → color) serves both purposes at
  once: "not in the map" = unvisited, "in the map" = visited with that color.
- **Flipping to the opposite color:** using `int` colors (`0`/`1`), XOR against
  `1` flips cleanly: `neighbourGroup = sourceGroup ^ 1`. (A `boolean`
  representation with `!color` works equally well — XOR isn't required, just a
  clean option.)
- **Disconnected graphs:** a single BFS from one node won't necessarily reach
  every node. Wrap the BFS in an outer loop over **all** nodes `0` to `A-1`
  (not just `adjList`'s keys — a node with zero edges may never become a key,
  but the loop still needs to consider it), only starting a new BFS if that
  node isn't already colored (`!groups.containsKey(node)`). This correctly
  handles each connected component independently.

## Algorithm
1. Build the adjacency list from edges (undirected — add both directions per
   edge, same pattern as [[find-if-path-exists-in-graph-bfs]]).
2. Outer loop `node = 0` to `A-1`: if `node` isn't colored yet, run `bfs(node)`.
   If any `bfs` call returns `false` (conflict found), return `0`/`false`
   immediately — no need to check remaining components.
3. Inside `bfs(node)`: standard BFS, but for each neighbor `i` of a dequeued
   node:
   - If `i` is **already colored**: check if its color matches the required
     `neighbourGroup`. If it doesn't match → conflict → return `false`
     immediately. If it matches → fine, do nothing further (don't re-enqueue
     an already-processed node — wasted work, though not incorrect).
   - If `i` is **uncolored**: assign it `neighbourGroup` and enqueue it (this
     is the step that actually keeps the BFS expanding — missing this means
     the BFS never gets past the starting node's direct neighbors).
4. If the outer loop completes with no conflicts found anywhere, return
   `1`/`true`.

## Complexity
- **Time: O(V + E)** — graph-building is O(E). The BFS+coloring traversal is
  the same O(V+E) as standard BFS (each node dequeued once, each edge examined
  twice total across all neighbor lists); the added color-conflict check per
  neighbor is O(1) (HashMap lookup), which doesn't change the complexity class.
- **Space: O(V + E)** — `adjList` holds O(E) entries (2 per edge). The `queue`
  holds at most O(V) nodes (it stores node IDs, not edges). `groups` holds at
  most O(V) entries — bounded by the number of distinct nodes that can ever be
  colored, regardless of edge count (e.g., a fully-connected graph with V=10,
  E=45 still only ever colors 10 nodes). Combined: O(V + E).

## Solution

```java
public class Solution {
    public int solve(int A, int[][] B) {
        Map<Integer,List<Integer>> adjList = new HashMap<>();

        for (int i = 0; i < B.length; i++) {
            int key = B[i][0];
            int value = B[i][1];
            if (adjList.containsKey(key)) {
                adjList.get(key).add(value);
            } else {
                List<Integer> list = new ArrayList<>();
                list.add(value);
                adjList.put(key, list);
            }
            int temp = key;
            key = value;
            value = temp;
            if (adjList.containsKey(key)) {
                adjList.get(key).add(value);
            } else {
                List<Integer> list = new ArrayList<>();
                list.add(value);
                adjList.put(key, list);
            }
        }

        Map<Integer,Integer> groups = new HashMap<>();

        for (int node = 0; node < A; node++) {
            if (!groups.containsKey(node))
                if (bfs(adjList, node, groups) == false) return 0;
        }
        return 1;
    }

    private boolean bfs(Map<Integer, List<Integer>> adjList, int node, Map<Integer,Integer> groups) {
        Queue<Integer> queue = new LinkedList<>();
        queue.add(node);
        groups.put(node, 1);

        while (!queue.isEmpty()) {
            int source = queue.remove();
            int sourceGroup = groups.get(source);
            int neighbourGroup = sourceGroup ^ 1;

            if (!adjList.containsKey(source)) {
                continue; // no edges from this node
            }
            List<Integer> neighbours = adjList.get(source);
            for (Integer i : neighbours) {
                if (groups.containsKey(i)) {
                    if (neighbourGroup != groups.get(i)) {
                        return false;
                    }
                } else {
                    queue.add(i);
                    groups.put(i, neighbourGroup);
                }
            }
        }
        return true;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Tried to iterate `for(Integer key : adjList)` — a `Map` directly | Maps aren't directly iterable; need `.entrySet()`, `.keySet()`, or (better here) loop over `0` to `A-1` directly, since a zero-edge node never becomes an `adjList` key at all |
| Used `groups.add(key, 1)` and `groups.getKey(i)` — neither is a real `Map` method | `Map` uses `.put(key, value)` to insert and `.get(key)` to retrieve — no `.add()` or `.getKey()` methods exist |
| Outer loop initially had no `!groups.containsKey(node)` guard — called `bfs` unconditionally for every node | Without the guard, `bfs` would forcibly recolor already-correctly-colored nodes on every subsequent call, risking corrupting valid colorings or introducing false conflicts |
| The critical bug: the `for(Integer i : neighbours)` loop had no `else` branch for uncolored neighbors in an early attempt | Without coloring+enqueueing uncolored neighbors, BFS never expands past the very first node's direct neighbors — silently produces wrong answers without throwing any error |
| `bfs` declared to return `boolean` but had no `return` statement in early drafts; `solve` had no `return` statement and didn't check `bfs`'s result | Every code path must return a value matching the declared return type; the outer loop needs to check each `bfs` call's result and short-circuit (`return 0`) the moment any component reports a conflict |
