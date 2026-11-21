# Network Delay Time (Dijkstra)

**Problem:** LeetCode 743. Given directed weighted edges `times[i] = [from, to, time]`, a number of cities `n` (labelled 1 to n) and a start city `k`, a signal starts at `k` and spreads along the roads. Return the time until every city has received it, or `-1` if some city can never be reached.

**Link:** https://leetcode.com/problems/network-delay-time/

**Pattern:** Dijkstra's shortest path (single source, non-negative weights). BFS with a min-heap instead of a plain queue.

---

## Core insight

- BFS counts the **fewest roads**. When roads have different times, the fewest roads is not the fastest route. Example: A to C directly takes 4, but A to B to C takes 1 + 2 = 3.
- Weights sit on the **edges** (roads), not on the nodes.
- Keep a **scoreboard**: best known time from the start to each city. Start city = 0, every other city = infinity.
- Repeatedly take the unprocessed city with the **smallest** known time (that is the job of a **min-heap**), look at its roads, and if going through it gives a neighbour a **strictly better** time, update the scoreboard **and push** that neighbour.
- One value per city in the scoreboard. The heap can hold **several entries for the same city**, because Java's PriorityQueue can't edit an entry. You push the better one and leave the old one.
- **Stale check:** right after a pop, if the popped time is greater than the scoreboard's value for that city, skip it. It is an outdated entry.
- Final answer: the signal spreads to all cities **in parallel**, so everyone has it when the slowest city does. Answer = **max** of the scoreboard. If any city is still infinity, return `-1`.
- Dijkstra needs **non-negative** edge weights.

## Plain-words plan

1. Build the adjacency list `Map<Integer, List<int[]>>`: from city -> list of `{to, time}`.
2. `score` of size n + 1 (index 0 unused): 0 for `k`, `Integer.MAX_VALUE` for the rest.
3. Min-heap of `{city, totalTimeFromStart}` ordered by time. Push `{k, 0}`.
4. While the heap isn't empty:
   - pop `{city, time}`
   - **stale check:** if `time > score[city]`, skip
   - for each road `{to, w}` out of `city`: `newScore = time + w`; if `newScore < score[to]`, set `score[to] = newScore` and push `{to, newScore}`
5. Final pass: any `MAX_VALUE` means return `-1`; otherwise return the max.

## Hand trace

Roads: 1->2 (1), 1->3 (4), 2->3 (2), 3->4 (1). Start k = 1.

| Step | Action | Scores (1,2,3,4) | Heap |
|---|---|---|---|
| start | | 0, inf, inf, inf | {1,0} |
| pop {1,0} | city 2 gets 0+1 = 1, city 3 gets 0+4 = 4 | 0, 1, 4, inf | {2,1} {3,4} |
| pop {2,1} | city 3: 1+2 = 3 < 4, update | 0, 1, 3, inf | {3,3} {3,4} |
| pop {3,3} | city 4: 3+1 = 4 | 0, 1, 3, 4 | {3,4} {4,4} |
| pop {3,4} | 4 > score[3] = 3, stale, skip | | {4,4} |
| pop {4,4} | no roads out | | empty |

Scores `0, 1, 3, 4`, max = 4. Answer 4.

## Complexity

V = cities, E = roads.

- **Time: O(V + E log E)**, which is the same as **O(E log V)**.
  - Building the adjacency list is O(E). The scoreboard and the final pass are O(V).
  - Each road can cause at most one push (only when it finds a strictly better time, and each city's roads are processed once because stale entries and ties are skipped). So at most E pushes, and the heap holds at most E entries.
  - One push or pop costs at most log E. Total E log E.
  - E is at most V² (V choices for the start of a road, V for the end), so log E is at most 2 log V. The 2 is a constant, so E log E is the same order as E log V.
- **Space: O(V + E).** Scoreboard V, adjacency list V keys plus E road arrays, heap up to E entries.

## Final code

```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        Map<Integer, List<int[]>> map = new HashMap<>();          // from -> list of {to, time}
        for (int i = 0; i < times.length; i++) {
            int[] arr = times[i];
            int source = arr[0];
            if (map.containsKey(source)) {
                List<int[]> list = map.get(source);
                list.add(new int[]{arr[1], arr[2]});
            } else {
                List<int[]> list = new ArrayList<>();
                list.add(new int[]{arr[1], arr[2]});
                map.put(source, list);
            }
        }

        int[] score = new int[n + 1];                             // index 0 unused
        for (int i = 1; i <= n; i++) {
            if (i == k) {
                score[i] = 0;
            } else {
                score[i] = Integer.MAX_VALUE;
            }
        }

        PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[1] - b[1]);   // {city, total time}
        minHeap.offer(new int[]{k, 0});

        while (minHeap.size() > 0) {
            int[] arr = minHeap.poll();
            if (arr[1] > score[arr[0]]) continue;                 // stale entry
            List<int[]> list = map.get(arr[0]);
            if (list == null) continue;                           // no roads out
            for (int[] a : list) {
                int newScore = a[1] + arr[1];                     // popped time + road time
                if (newScore >= score[a[0]]) continue;            // not strictly better
                score[a[0]] = newScore;
                minHeap.offer(new int[]{a[0], newScore});         // push a NEW pair, never the road itself
            }
        }

        int max = Integer.MIN_VALUE;
        for (int i : score) {
            if (i == Integer.MAX_VALUE) {
                return -1;                                        // some city never reached
            }
            if (i > max) {
                max = i;
            }
        }
        return max;
    }
}
```

## Mistakes and notes

| Item | What happened | Fix / rule |
|---|---|---|
| When to use Dijkstra | Said "when nodes have a weight" | The weight is on the **edges**. Use Dijkstra for the cheapest total over weighted edges |
| Two values for one city | Asked why C would have two values | Scoreboard: one value per city. Heap: may hold stale extra entries, because PriorityQueue can't edit in place |
| Adjacency value | Said the value is "an integer array" | Value is a **list** of `{to, time}` arrays, since a city has many roads out |
| New time | Said update with "the road's time" | New time = **popped time + road time** |
| Final answer | Said sum of the scores (0 + 1 + 3 + 4) | Scores are totals from the start and the signal spreads in parallel: take the **max** |
| Final pass in code | Used `min` and skipped zeros | `max`. Return `-1` if any score is still infinity |
| Changed the stored road | `a[1] = a[1] + arr[1]` overwrote the road's time | Use a local `newScore`. Never modify the adjacency list during the search |
| Pushed the road | `minHeap.offer(a)` put `{city, roadTime}` in the heap | Push `new int[]{a[0], newScore}`. The heap needs total time from the start |
| Stale check | Left out at first | Right after poll: `if (popped time > score) continue;` |
| Ties | `newScore > score` let equal routes through | `>=` skips anything not strictly better. Ties cause repeated processing |
| Plan vs code | Plan was right, code pushed the wrong thing | Before pasting a heap problem, check what exactly is inside each pushed entry |
| Complexity | Gave V + E + E log E | Correct. Simplify to O(V + E log E), and O(E log V) since E is at most V squared |
