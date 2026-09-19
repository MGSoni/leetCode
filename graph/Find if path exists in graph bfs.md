# Find if Path Exists in Graph

**Pattern:** Graph Traversal — BFS

## Problem
Given `n` nodes labeled `0` to `n-1`, and an undirected graph given as `edges`
(each `edges[i] = [a, b]`), determine if there is a valid path from `source` to
`destination`.

LeetCode 1971: https://leetcode.com/problems/find-if-path-exists-in-graph/

## Mental model — Prefixo the hiker (BFS intuition)
BFS explores level by level: all direct neighbors first, then their neighbors,
and so on — like exploring "everyone within 2 degrees of separation" before
going further out.
- Use a **Queue** (FIFO) to track "what to explore next" — matches the natural
  level-by-level order (add newly discovered nodes to the back, explore from
  the front).
- Use a **HashSet** for O(1) "have I seen this before?" lookups.
- **Mark a node visited at the moment you enqueue it, not when you dequeue it.**
  Marking only at dequeue time allows the same node to be enqueued multiple
  times before it's ever processed (if two different nodes both discover it
  before either is dequeued).

## Approach
1. **Build the adjacency list** from `edges`. Since the graph is undirected,
   each edge `[a, b]` must be added in **both** directions: `b` into `a`'s
   neighbor list, and `a` into `b`'s neighbor list.
   - Standard check-then-create pattern per direction:
     ```java
     if (adjList.containsKey(a)) {
         adjList.get(a).add(b);
     } else {
         List<Integer> valueList = new ArrayList<>();
         valueList.add(b);
         adjList.put(a, valueList);
     }
     ```
   - Key insight: once you have a list reference (new or fetched via `.get()`),
     modifying it directly (`.add(...)`) updates the same object already sitting
     in the map — no need to call `.put()` again afterward.
2. **Edge case:** guard `if (source == destination) return true;` up front —
   also protects against `adjList.get(source)` being null if `source` never
   appears in any edge.
3. **BFS:** seed `queue` and `visitedSet` with `source`. While the queue isn't
   empty: dequeue a node, iterate its neighbors, check each against
   `destination` (return `true` on match), and enqueue+mark any unvisited
   neighbor.
4. If the queue empties without ever reaching `destination`, return `false`.

## Complexity
- **Time: O(V + E)** — the graph-building loop is O(E) (one pass over edges).
  In the BFS itself: the outer `while` loop dequeues each node at most once
  (O(V)); the inner `for` loop's total work across *all* dequeues equals the
  sum of every neighbor list's size. For an undirected graph, each edge
  contributes exactly 2 entries total across all neighbor lists (one per
  endpoint) — so that sum is `2E`, i.e., O(E). Combined: O(V + E). This is the
  standard graph traversal complexity, worth having memorized cold.
- **Space: O(V + E)** — the adjacency list has `V` keys, but its neighbor lists
  collectively hold `2E` entries → O(E) for that alone. `visitedSet` and
  `queue` each scale with O(V). Combined: O(V + E).

## Solution

```java
class Solution {
    public boolean validPath(int n, int[][] edges, int source, int destination) {
        if (source == destination) return true;
        Map<Integer, List<Integer>> adjList = new HashMap<>();

        for (int i = 0; i < edges.length; i++) {
            int a = edges[i][0];
            int b = edges[i][1];

            if (adjList.containsKey(a)) {
                adjList.get(a).add(b);
            } else {
                List<Integer> valueList = new ArrayList<>();
                valueList.add(b);
                adjList.put(a, valueList);
            }

            if (adjList.containsKey(b)) {
                adjList.get(b).add(a);
            } else {
                List<Integer> valueList = new ArrayList<>();
                valueList.add(a);
                adjList.put(b, valueList);
            }
        }

        Set<Integer> visitedSet = new HashSet<>();
        Queue<Integer> queue = new LinkedList<Integer>();

        queue.add(source);
        visitedSet.add(source);

        while (!queue.isEmpty()) {
            int top = queue.remove();
            List<Integer> neighbours = adjList.get(top);
            for (int i : neighbours) {
                if (i == destination) {
                    return true;
                }
                if (!visitedSet.contains(i)) {
                    queue.add(i);
                    visitedSet.add(i);
                }
            }
        }
        return false;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Used `Queue.push()`/`.pop()` (Stack/Deque method names, LIFO) on a `Queue<Integer>` | `Queue`'s FIFO methods are `.add()`/`.offer()` (enqueue) and `.remove()`/`.poll()` (dequeue) — pushing to a queue with the wrong method names doesn't match the intended traversal order |
| Wrote `if (i = destination)` — assignment instead of comparison | Use `==` for equality comparison, not `=` (assignment). Also asked: does Java even allow `if(i = destination)` to compile for `int` types? It doesn't — Java requires the `if` condition to be a `boolean`, unlike C where this is legal-but-dangerous |
| Forgot the reverse-direction graph edge (`b→a`) initially — graph was effectively directed | LC 1971 is undirected: every edge `[a,b]` must be added to *both* `a`'s and `b`'s neighbor lists |
| Initial attempts mixed up list-creation and list-modification into single broken statements (e.g. `new valueList.add(b)`) | Break multi-step logic into separate, explicit statements: create the list, put it in the map, then add to it — don't try to compress multiple operations into one invalid expression |
| Assumed TC/SC scale with the *for-loop-over-neighbours* as `E*N` (edges × nodes) | Traced a concrete small example and found each edge contributes exactly 2 entries total across all neighbor lists combined (once per endpoint) — so total inner-loop work across the entire BFS is `2E` = O(E), not O(E·N). Standard graph traversal complexity is O(V+E) for both time and space. |
