# Number of Provinces (Union-Find)

**Problem:** LeetCode 547. Given an n x n matrix `isConnected` where `isConnected[i][j] == 1` means city i and city j are directly connected, return the number of provinces (groups of cities connected directly or indirectly).

**Link:** https://leetcode.com/problems/number-of-provinces/

**Pattern:** Union-Find (Disjoint Set Union). Counting connected components when the input is a connection table, not a picture.

---

## Core insight

- The matrix is a **connection table**, not a grid picture. Node = city, neighbor = column j where `isConnected[i][j] == 1`. Island-style flood fill gives the wrong answer (5 instead of 2 on `[[1,0,1],[0,1,0],[1,0,1]]`).
- Start with n groups, one per city. Every time a connection joins two **different** groups, merge them and do `count--`.
- Same leader means already in one group: skip, do not subtract.
- The table tells us which cities are connected. The parent array is our record of which groups we know about. We merge when the table and the record disagree.

## Plain-words plan

1. `parent[i] = i` for every city, `count = n`.
2. For every pair (i, j) with `i != j` and `isConnected[i][j] == 1`:
   - find the leader of i and the leader of j
   - if the leaders differ: point one **leader** at the other, `count--`
   - if the leaders are the same: do nothing
3. Return `count`.

## find (with path compression)

- Walk 1: follow parents from the city until a city points to itself. That city is the leader.
- Walk 2: walk again from the original city, and set each city's parent directly to the leader. Save the current city in `temp` before moving up, otherwise the chain is lost.
- Effect: the next find on the same city takes one step.

## Hand trace

Example 1: `[[1,1,0],[1,1,0],[0,0,1]]`

| Step | Action | parent | count |
|---|---|---|---|
| Start | each city its own leader | [0,1,2] | 3 |
| i=0, j=1 | leaders 0 and 1 differ, parent[1] = 0 | [0,0,2] | 2 |
| i=1, j=0 | leaders 0 and 0 same, skip | [0,0,2] | 2 |
| i=2 | only connected to itself | [0,0,2] | 2 |

Answer: 2.

Chain example for path compression: parent = [0,0,1,2] (0 <- 1 <- 2 <- 3). find(3): leader is 0. Second walk sets 3 to 0, 2 to 0, 1 to 0, so parent = [0,0,0,0].

## Complexity

- **Time:** O(n^2). The double loop visits n x n cells, and each find is almost constant after path compression (amortized, with a tiny inverse-Ackermann factor). Without path compression the worst case is O(n^3), since each find can walk a chain of length n.
- **Space:** O(n) for the parent array. The input matrix is given, not extra.

## BFS/DFS vs Union-Find

- BFS/DFS: walk the map. Explore from a city, mark everything reachable, count groups by exploring.
- Union-Find: keep a record of who belongs to which group, update it for each fact "i and j are connected".
- Table given up front (this problem): all three cost about O(n^2).
- Connections arriving one at a time, or many "are these two connected?" queries: Union-Find wins, because each update is cheap and the answer is always ready. BFS/DFS would restart after every new connection.
- Later problems where this shows up: Redundant Connection, Accounts Merge, Kruskal's algorithm.

## Final code

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;
        int m = isConnected[0].length;

        int[] parentArray = new int[n];
        int count = n;
        for (int i = 0; i < n; i++) {
            parentArray[i] = i;
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (i != j && isConnected[i][j] == 1) {
                    int iLeader = helper(i, parentArray);
                    int jLeader = helper(j, parentArray);
                    if (iLeader != jLeader) {
                        parentArray[jLeader] = iLeader;
                        count--;
                    }
                }
            }
        }
        return count;
    }

    private int helper(int node, int[] parentArray) {
        int leader = node;
        while (parentArray[leader] != leader) {
            leader = parentArray[leader];
        }

        while (parentArray[node] != node) {
            int temp = node;
            node = parentArray[node];
            parentArray[temp] = leader;
        }
        return leader;
    }
}
```

## Mistakes to remember

| Mistake | What happened | Fix |
|---|---|---|
| Array holds connected 0/1 | Thought the parent array stores "connected or not" | Parent array stores each city's **leader**, starts as `parent[i] = i` |
| find loops over whole array | Described find as scanning all cities | find follows one chain from the given city only |
| Flipped skip/merge rule | Said different leaders means no merge | Same leader = skip. Different leaders = merge and `count--` |
| `parentArray[j] = iLeader` | Overwrote city j, cutting it from its old group | Point the **leader**: `parentArray[jLeader] = iLeader` |
| Compression missing | Thought merge line was compression | Compression is in find: second walk sets each city's parent to the leader |
| Complexity wording | Called find "constant" | Amortized near-constant with compression, O(n^3) worst case without |
