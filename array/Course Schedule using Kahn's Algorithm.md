# Course Schedule using Kahn's Algorithm

**Pattern:** BFS topological sort (indegree counting) for cycle detection

## Problem (Scaler version)
There are `A` courses labeled `1` to `A`. Two arrays `B` and `C` of the same length
describe prerequisite pairs: pair `i` is `(B[i], C[i])`, meaning `B[i]` must be taken
**before** `C[i]`. Return `1` if all courses can be finished, `0` otherwise.

This is the same question as LC 207 ([[course-schedule-cycle-detection-dfs]]), solved
with BFS instead of DFS. Kahn's also gives a valid course order, so the same technique
solves LC 210 ([[course-schedule-ii-topological-sort-dfs]]).

## Core idea
Give every course a counter, its **indegree**: how many prerequisites it is still
waiting on. A course whose counter is `0` can be taken right now.

1. Put every course with counter `0` into one queue.
2. Pop a course and count it as taken.
3. For each course that was waiting on it, lower that course's counter by 1.
4. If a counter just reached `0`, push that course.
5. Repeat until the queue is empty.
6. If you took all `A` courses, return `1`. If you took fewer, return `0`.

**Why a cycle is caught:** courses in a cycle wait on each other, so their counters never
reach `0`, they are never pushed, and the number of courses taken stays below `A`. No
`visited`/`inProgress` sets are needed.

## Key rule: push only when the counter reaches 0
Lowering a counter does not mean push. Take `B = [1, 2]`, `C = [3, 3]` (course 3 needs
both 1 and 2). After taking course 1, course 3's counter is 2 - 1 = 1. It is still
waiting on course 2, so it is not pushed yet. After taking course 2, the counter hits 0
and course 3 is pushed once. A counter reaches 0 exactly once, so every course is pushed
at most once and no duplicate pushes can happen.

## Edge direction
- Kahn's needs **prerequisite -> course**: `B` is the key and the list of `C` values is
  the value. When you take a course, the map answers "who was waiting on this?"
- The DFS finish-order method for LC 210 needed the opposite, course -> prerequisite.
  Direction depends on the algorithm, so re-derive it each time.
- Here `B` and `C` are two parallel arrays, so pair `i` is `(B[i], C[i])`. In LC 207
  each pair was one row, `pr[i][0]` and `pr[i][1]`.

## Counter storage
Use `int[] counter = new int[A + 1]`. Courses are labeled `1` to `A`, so size `A + 1`
lets the label be the index, and slot `0` is unused. When seeding, skip index `0`. Slots
start at `0`, so a course nobody points at already reads as counter 0. A map would work
too, but a course that never appears in `C` would have no key, so you would have to loop
over `1..A` with `getOrDefault`.

The counters are counted over `C` only: each appearance of a course in `C` is one
prerequisite pointing at it.

## Hand traces
- **Example 1** (`A=3`, `B=[1,2]`, `C=[2,3]`): map `{1:[2], 2:[3]}`, counters 1:0, 2:1,
  3:1. Queue `[1]`. Take 1, so counter of 2 becomes 0 and 2 is pushed. Take 2, so counter
  of 3 becomes 0 and 3 is pushed. Take 3. Courses taken = 3 = A, so return `1`.
- **Example 2** (`A=2`, `B=[1,2]`, `C=[2,1]`): counters 1:1, 2:1. No counter is 0, so the
  queue starts empty. Courses taken = 0 != 2, so return `0`.
- **Shared prerequisite** (`B=[1,1]`, `C=[2,3]`): map `{1:[2,3]}`, counters 1:0, 2:1, 3:1.
  Take 1, so both 2 and 3 reach 0 and are both pushed. Courses taken = 3.

## Complexity
(Check these against your own derivation. Here V = A courses and E = number of pairs.)
- **Time: O(A + E).** Building the map and the counters is O(E). Seeding scans the counter
  array, O(A). In the BFS each course is popped at most once, and each pair is examined
  once when its prerequisite is popped.
- **Space: O(A + E).** The map holds up to E values, the counter array is O(A), and the
  queue holds at most O(A) courses.

## Solution

```java
public class Solution {
    public int solve(int A, int[] B, int[] C) {

        Map<Integer,List<Integer>> map = new HashMap<>();
        for (int i = 0; i < B.length; i++) {
            int key = B[i];
            int value = C[i];
            if (map.containsKey(key)) {
                List<Integer> list = map.get(key);
                list.add(value);
            } else {
                List<Integer> list = new ArrayList<>();
                list.add(value);
                map.put(key, list);
            }
        }

        int[] counter = new int[A + 1];
        for (int i = 0; i < C.length; i++) {
            counter[C[i]]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < counter.length; i++) {
            if (i > 0 && counter[i] == 0) {
                queue.add(i);
            }
        }

        int courseTaken = 0;

        while (!queue.isEmpty()) {
            int preReq = queue.remove();
            courseTaken++;
            List<Integer> courses = map.get(preReq);
            if (courses == null) {
                continue;
            }
            for (int course : courses) {
                counter[course]--;
                if (counter[course] == 0) {
                    queue.add(course);
                }
            }
        }

        if (courseTaken == A) return 1;

        return 0;
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| First plan reused the DFS approach with `visited` and `inProgress` sets | That is the LC 207 technique. Kahn's replaces both sets with indegree counters and a queue |
| Proposed `C` as the key and `B` as the value (course -> its prerequisites) | Kahn's needs prerequisite -> course, so `B` is the key and `C` is the value. The map must answer "who was waiting on this course?" |
| Read `B` and `C` as one structure ("`B[0]` and `C[0]` are keys, `B[1]` and `C[1]` are values") | They are parallel arrays: pair `i` is `(B[i], C[i])` |
| Counted the counters over both arrays | Count over `C` only. Each appearance in `C` is one prerequisite pointing at that course |
| Said taking course 1 lowers the counters of both 2 and 3 in a 1 -> 2 -> 3 chain | Taking a course only lowers the counters of the courses that depend on it directly |
| Planned a separate BFS call for each zero-counter course | Seed one queue with all zero-counter courses and run a single BFS |
| Planned to push an unlocked course as soon as its counter was lowered | Push only when the counter reaches 0, otherwise a course with several prerequisites is pushed too early |
| Confused "courses with counter 1" with the final check | The check is courses taken vs `A`. A plain `int` counter of pops is enough, so no set is needed |
