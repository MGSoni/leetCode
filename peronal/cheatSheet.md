# DSA Pattern Recognition Cheat Sheet

How to look at an unfamiliar question (for example, a Google-style one) and decide
which pattern to try first. Interview questions are usually a known pattern in
disguise, so the skill is matching the **structure**, not the story.

## Step 1: a 60-second routine before you pick anything
1. **Restate the input and the output.** Is the input an array, a string, a grid, a
   list of pairs (a graph in disguise), a tree, or a stream?
2. **Read the constraints.** They tell you the target complexity (table below).
3. **Find the signal words** in the question (table below).
4. **Say the brute force out loud**, then ask: what work is being repeated, and what
   structure could remember it? (a hashmap, a prefix sum, a visited set, a heap, a DP table)
5. **Pick the pattern, write pseudocode, hand-trace a tiny example, then code.**

## Step 2: constraints tell you the target complexity
| Input size | Target | Usually means |
|---|---|---|
| n <= 20 | O(2^n), O(n!) | Backtracking, subsets, permutations, bitmask |
| n <= 100 | O(n^3) | Triple loops, small DP |
| n <= 1,000 to 5,000 | O(n^2) | Nested loops, 2D DP, grid problems |
| n <= 100,000 | O(n log n) or O(n) | Sorting, heap, binary search, two pointers, sliding window, hashing, BFS/DFS |
| n >= 1,000,000 | O(n) or O(log n) | One pass, math, binary search on a formula |

## Step 3: signal words to pattern
Entries marked **(covered)** are patterns we practiced in this chat. The others are
standard patterns we haven't covered here yet.

| If the question says... | Think... | First thing to ask yourself | Anchor problem |
|---|---|---|---|
| "number of subarrays with sum equal to k" | **Prefix sum + hashmap** (covered) | What earlier prefix value completes the sum? Count or longest? | Subarray Sum = K |
| "longest subarray with equal 0s and 1s" or "sum zero" | **Prefix sum + hashmap, first index** (covered) | Can I turn the condition into "sum equals 0" (0 becomes -1)? | Contiguous Array |
| "find triplets/pairs that sum to X", sorted or sortable | **Sort + two pointers** (covered) | Which pointer moves when the sum is too small or too big? How do I skip duplicates? | 3Sum |
| "all combinations / subsets / permutations / arrangements" | **Backtracking** (covered) | What is one choice? What is the base case? What do I undo? | Letter Combinations |
| string is a rotation / repeated pattern | **s + s trick** (covered) | Is every rotation a substring of the doubled string? | Rotate String |
| "can you reach X from Y", "is there a path", "connected" | **BFS or DFS** (covered) | Are the edges directed or undirected? | Path Exists in Graph |
| "minimum number of steps/moves/minutes" with equal-cost moves | **BFS, count levels** (covered) | What is a state? Is it one start or many? | Rotting Oranges |
| "count regions / islands / connected groups" in a grid | **Grid DFS or BFS** (covered) | Where do I mark visited? Do I need the size or only the count? | Number of Islands, Max Area |
| "enclosed / surrounded / touches the border" | **Two-pass flood fill from the border** (covered) | Can I remove the border-connected cells first? | Closed Islands |
| "flows to / reaches both X and Y" | **Search backward from the targets, intersect** (covered) | Is it cheaper to search from the destinations than from every cell? | Pacific Atlantic |
| "can the items be split into two groups with no conflict" | **2-coloring with BFS** (covered) | Does a colored neighbor ever conflict? | Is Graph Bipartite |
| "dependencies / prerequisites / ordering" | **Topological sort** (covered) | Is there a cycle? Do I need the order or only yes/no? | Course Schedule I and II |
| "is there a cycle in this directed graph" | **DFS with visited vs in-progress, or Kahn's** (covered) | Am I revisiting a finished node (fine) or an active ancestor (cycle)? | Course Schedule |
| "top K / K-th largest / smallest / most frequent" | **Heap** (not covered yet) | Min-heap of size K or sort everything? | Top K Frequent Elements |
| "merge K sorted lists / streams" | **Heap** (not covered yet) | What goes into the heap, and what do I push after each pop? | Merge K Sorted Lists |
| "autocomplete / prefix search / words with a common prefix" | **Trie** (not covered yet) | What is stored at each node? | Implement Trie |
| "shortest path with weights" | **Dijkstra** (not covered yet) | Are all weights non-negative? | Network Delay Time |
| "are X and Y in the same group", groups merge over time | **Union-Find** (not covered yet) | What does find() return, and what does union() merge? | Number of Provinces |
| "minimum / maximum / count ways" with overlapping subproblems | **Dynamic programming** (earlier prep) | What is the state? What are the choices? What is the base case? | Coin Change, LCS |
| "next greater element", "span", "histogram" | **Monotonic stack** (earlier prep) | Should the stack be increasing or decreasing? | Next Greater Element |
| "minimum capacity / speed / time such that ..." | **Binary search on the answer** (earlier prep) | Is feasibility monotonic (once true, always true)? | Koko Eating Bananas |
| "longest / shortest window with a property" | **Sliding window** (earlier prep) | What makes the window valid, and when do I shrink it? | Longest Substring Without Repeating |
| "overlapping intervals / meeting rooms" | **Sort by start + sweep** (earlier prep) | What do I compare with the last interval? | Merge Intervals |

## Graph problems hide in many costumes
You may never see the word "graph". Treat it as a graph when you see:
- **A grid:** each cell is a node and its 4 neighbors are the edges.
- **A list of pairs:** `[a, b]` meaning "a needs b", "a and b are friends", "a connects to b".
- **States that change by one move:** a lock combination, a word changing one letter at a time, a puzzle board. Each state is a node and each legal move is an edge (Open the Lock, Word Ladder).
- **Dependencies or ordering rules:** that points to a directed graph and topological sort.

## BFS or DFS?
| Use BFS when... | Use DFS when... |
|---|---|
| You need the **shortest** path or minimum steps (equal cost per move) | You need to explore **everything** in a region (count, size, flood fill) |
| Things spread from **many sources at once** | You need cycle detection or finish order |
| Recursion depth could overflow the stack | The recursive structure is natural (trees, backtracking) |

## Traps to check before you code
- **Direction:** undirected means add both directions. Directed means decide the direction from what the algorithm needs.
- **Visited timing:** BFS marks at push. DFS marks before recursing.
- **Labels:** are nodes numbered from 0 or from 1? That changes your array size.
- **Disconnected input:** loop over every node and start a search only from unvisited ones.
- **Empty or degenerate input:** `[]`, one cell, `source == destination`.
- **Count or longest?** A hashmap storing counts and one storing first indexes solve different questions.
- **Complexity:** state it yourself, naming V and E, or m and n.

## If nothing matches
1. Solve a tiny example by hand and watch what you do. That is usually the algorithm.
2. Write the brute force and find what it recomputes.
3. Ask: can I sort it, hash it, or search the answer space?
4. Say your reasoning out loud. Interviewers score the thinking, not only the final code.
