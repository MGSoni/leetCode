# Find Eventual Safe States

**Pattern:** Directed-graph cycle detection, DFS with a stamp (verdict) per node

## Problem
A directed graph has `n` nodes labeled `0` to `n-1`, given as `graph[i]`, the list of nodes
that `i` points to. A node is **terminal** if it has no outgoing edges. A node is **safe** if
**every** path starting at it ends at a terminal node. Return all safe nodes in ascending order.

Example: `graph = [[1,2],[2,3],[5],[0],[5],[],[]]` -> `[2,4,5,6]`

LeetCode 802: https://leetcode.com/problems/find-eventual-safe-states/

## Core insight
- A node is **safe** if **all of its neighbors are safe**. A terminal node has no neighbors, so it
  is safe by definition.
- A node is **unsafe** if it is on a cycle **or can reach a cycle**. One bad path is enough.
  Example: in `[[1],[1],[]]`, node 0 is not on any cycle, but its only path goes into the
  self-loop at node 1, so node 0 is unsafe.
- You need a verdict for **every** node, not one yes/no for the whole graph. That is the
  difference from Course Schedule ([[course-schedule-cycle-detection-dfs]]), where one cycle
  anywhere settles the answer.

## One array, three stamps
`stamp[node]` starts at `0`.
- `0` = not visited yet
- `1` = being explored right now, or proven unsafe
- `2` = proven safe

A node that fails keeps stamp `1`. Reaching it again also means "not safe", so no separate
"unsafe" state is needed. The recursion itself is the current path, so no explicit stack is
needed.

## Algorithm
```
dfs(node):                         // answers: "is this node safe?"
    if stamp[node] == 1: return false      // loop found, or already known unsafe
    if stamp[node] == 2: return true       // already known safe
    stamp[node] = 1                        // exploring it now
    for each neighbor in graph[node]:
        if dfs(neighbor) == false: return false   // leave the stamp at 1
    stamp[node] = 2                        // all neighbors safe, so this node is safe
    return true

main: for node = 0 .. n-1: dfs(node)
answer: every node whose stamp is 2, scanned in index order (already ascending)
```
- `graph` already is an adjacency list. Do not build another one.
- Scanning `stamp` in index order gives an ascending answer, so no sorted set is needed.
- The verdict is stored **by the DFS, per node, as it unwinds**. If only the start node were
  marked, the other nodes on the bad path would be explored again and again.

## Hand traces
**`[[1,2],[2,3],[5],[0],[5],[],[]]`:**
- `dfs(0)` -> `dfs(1)` -> `dfs(2)` -> `dfs(5)` (no neighbors, stamp 2). Node 2 gets stamp 2.
- Back in node 1, neighbor 3: `dfs(3)` goes to node 0, which has stamp 1, so it returns false.
  That makes 3, then 1, then 0 return false. Nodes 0, 1 and 3 stay at stamp 1.
- Node 4 sees node 5 already at stamp 2, so it gets stamp 2. Node 6 has no neighbors, so stamp 2.
- Final stamps: 0:1, 1:1, 2:2, 3:1, 4:2, 5:2, 6:2 -> answer `[2,4,5,6]`.

**`[[1],[1],[]]`:** stamps end at 0:1, 1:1, 2:2 -> answer `[2]`.

## Why not a forward BFS
Starting a BFS from node 0 and spreading level by level gets confusing here, because node 1 has
one good path and one bad path and the BFS has no place to carry that verdict back. The
direction of the information is the reason:
- A forward BFS spreads **outward** from a start node.
- A node's verdict depends on the nodes it points to, so it flows **backward**, from the ends of
  the paths toward the start.
- Recursion that returns a value fits this well: each call asks its neighbors and gets
  true/false back.

**Rule to remember:** if a node's answer depends on the answers of the nodes it points to, use
recursion that returns a value, or start from the nodes whose answer is already known and work
backward.

## Alternative: Kahn's algorithm on the reversed graph (second rep, not done yet)
Start from terminal nodes (safe by definition). Reverse every edge. Each node keeps a counter of
outgoing edges that are still unresolved. When a neighbor becomes safe, decrement the counter, and
when it reaches 0 the node is safe and goes into the queue. Same technique as
[[course-schedule-kahns-algorithm-bfs]], applied to the reversed graph.

## Complexity
To be filled in after we discuss it. Work these out yourself first:
- **Time:** use the stamps to explain why each node's neighbor loop runs at most once. What does
  the total come to in terms of V (nodes) and E (edges)?
- **Space:** what is the deepest the recursion can go? Which structures do you count: the `stamp`
  array, the result list, the input `graph`?

## Solution

```java
class Solution {
    public List<Integer> eventualSafeNodes(int[][] graph) {
        int n = graph.length;

        int stamp[] = new int[n];

        for (int i = 0; i < n; i++) {
            dfs(i, graph, stamp);
        }
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (stamp[i] == 2) {
                result.add(i);
            }
        }
        return result;
    }

    private boolean dfs(int node, int[][] graph, int[] stamp) {
        if (stamp[node] == 1) return false;
        if (stamp[node] == 2) return true;

        stamp[node] = 1;
        for (int i = 0; i < graph[node].length; i++) {
            if (dfs(graph[node][i], graph, stamp) == false) return false;
        }
        stamp[node] = 2;
        return true;
    }
}
```

## Mistakes and confusions while working through it

| What went wrong | Fix / insight |
|---|---|
| Pictured the paths from a node as a tree that ends at terminal nodes | A graph can loop back (`0 -> 1 -> 3 -> 0`), so the "tree" never finishes. A path that can loop forever never reaches a terminal node |
| Planned to mark only the node where the search started when a cycle was found | Every node on the bad path is unsafe. The DFS must store the verdict itself, for each node, as it unwinds. Otherwise those nodes are explored repeatedly |
| Treated the question as "is this node on a cycle" | A node is also unsafe if it can **reach** a cycle. Node 0 in `[[1],[1],[]]` is not on a cycle but is unsafe |
| Thought an adjacency list had to be built | `graph[i]` already is one. Read it directly |
| Proposed a visited stack or a sorted set | One `int[]` of stamps is enough, and scanning it in index order gives ascending output |
| Tried to solve it with a forward, level-by-level BFS | The verdict flows backward from the path ends. Use value-returning recursion, or Kahn's on the reversed graph |

**Final code had no bugs.** The plan was written as pseudocode and hand-traced on two examples before the Java.
