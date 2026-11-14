# Redundant Connection (Union-Find)

**Problem:** LeetCode 684. A graph started as a tree with n nodes (labeled 1 to n), then one extra edge was added. Given `edges` (n edges), return the edge that can be removed so the graph is a tree again. If several answers exist, return the one that appears last in the input.

**Link:** https://leetcode.com/problems/redundant-connection/

**Pattern:** Union-Find, cycle detection. Process edges in order; the edge whose two ends already share a leader closes the cycle.

---

## Core insight

- Start with every city as its own leader.
- Edge arrives with **different** leaders: the groups are separate, so merge them (point one leader at the other).
- Edge arrives with the **same** leader: both ends are already connected, so this edge closes a cycle. Return it.
- Why the answer is always found: n cities need only n - 1 useful edges to be one group, and we are given n edges. So at most n - 1 merges can happen, and at least one edge must arrive with the same leader. The loop can never finish without finding it.
- Why the *first* such edge is also the *last-in-input* answer: the problem has exactly one extra edge, so the first same-leader edge is the only one. (Example: `[1,2],[2,3],[3,1],[4,5]` returns `[3,1]`, and `[4,5]` is never reached.)
- Cities are numbered from 1, so the parent array has size n + 1 (index 0 unused). n is `edges.length`.

## Plain-words plan

1. `n = edges.length`. Make `parent` of size n + 1 with `parent[i] = i`.
2. For each edge `{a, b}` in order:
   - `leaderA = find(a)`, `leaderB = find(b)`
   - same leader: return this edge
   - different: `parent[leaderB] = leaderA`
3. After the loop: unreachable placeholder return (Java needs one).

## Hand trace

Example: `[[1,2],[2,3],[3,1],[4,5]]`

| Edge | Leaders on arrival | Action | parent (index 1 to 5) |
|---|---|---|---|
| start | | | [1,2,3,4,5] |
| 1-2 | 1, 2 differ | parent[2] = 1 | [1,1,3,4,5] |
| 2-3 | 1, 3 differ | parent[3] = 1 | [1,1,1,4,5] |
| 3-1 | 1, 1 same | return [3,1] | |

Example A: `[[1,2],[2,3],[3,4],[1,4]]`. The first three edges merge (all leaders differ), and `1-4` arrives with both ends under leader 1, so the answer is `[1,4]`.

## Complexity

- **Time:** O(n). One loop over n edges, and each find is amortized near-constant with path compression. Strictly O(n x alpha(n)), where alpha is the inverse Ackermann function (below 5 for any real input).
- **Space:** O(n) for the parent array of size n + 1. The input edges are given, not extra.

## Final code (clear version)

```java
class Solution {
    public int[] findRedundantConnection(int[][] edges) {
        int n = edges.length;
        int[] parent = new int[n + 1];          // cities are 1..n, index 0 unused
        for (int i = 1; i <= n; i++) {
            parent[i] = i;                      // everyone starts as their own leader
        }

        for (int[] edge : edges) {
            int leaderA = find(edge[0], parent);
            int leaderB = find(edge[1], parent);

            if (leaderA == leaderB) {
                return edge;                    // both ends already connected: this edge closes the cycle
            }
            parent[leaderB] = leaderA;          // write to the LEADER, not to the city
        }
        return new int[0];                      // unreachable: n edges always contain a cycle
    }

    private int find(int node, int[] parent) {
        int leader = node;
        while (parent[leader] != leader) {      // walk 1: find the leader
            leader = parent[leader];
        }
        while (parent[node] != node) {          // walk 2: path compression
            int next = parent[node];
            parent[node] = leader;
            node = next;
        }
        return leader;
    }
}
```

### Your version (accepted, same logic)

Your solution filled a `result` array and used `break`, then returned `result`. It is correct. The version above only replaces that with `return edge;` inside the loop, which is shorter.

## Union-Find template (reuse in later problems)

- `parent[i] = i` at start.
- `find(x)`: walk to the leader, then compress the path.
- `union(a, b)`: `la = find(a)`, `lb = find(b)`; if different, `parent[lb] = la`.
- The same-leader check is the question "are these already connected?". Provinces uses it to decide whether to `count--`, and Redundant Connection uses it to detect the cycle.

## Mistakes and notes

| Item | What happened | Fix / rule |
|---|---|---|
| Reason for "never finishes" | Said it can't finish without finding the edge "because it's a tree" | A tree has no cycle. The real reason is counting: n edges, at most n - 1 merges |
| Order inside the loop | Said "check if the edges differ" before getting leaders | First call find on both cities, then compare leaders |
| Self-loop check | Considered checking `a != b` | Not needed: the problem has no edge from a city to itself |
| Merge write | Wrote to the leader (`parent[leaderB] = leaderA`) | Habit from the Provinces bug is now correct. Keep it |
| Return style | Filled `result` then `break` | `return edge;` directly is shorter. Placeholder return stays at the end |
| Complexity | O(n) time and space, stated the amortized assumption | Strict bound is O(n x alpha(n)) |
