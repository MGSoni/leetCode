# DSA Pattern Recognition Cheat Sheet (full catalog)

How to look at an unfamiliar question (for example, a Google-style one) and decide
which pattern to try first. Interview questions are usually a known pattern in
disguise, so the skill is matching the **structure**, not the story.

**How to read the tables**
- **Status:** `Chat` = we practiced it in this chat. `Earlier` = you told me you covered it in
  earlier prep (check it still holds). `To learn` = not covered in this chat, verify yourself.
- `Chat 2` = practiced in the second chat (recursion, backtracking, trees, linked lists, DP). A value like `Earlier + Chat 2` means both apply.
- **Priority:** my own judgement for senior backend interviews (High, Medium, Low). It is not
  survey data. Use it to decide the study order, not as a rule.

---

## Part 1: Master list of pattern names, by level

**Basic**
Hashing / frequency count, two pointers, fixed sliding window, prefix sum, sorting then scan,
standard binary search, simple recursion, stack for matching (parentheses), queue simulation,
linked list pointer manipulation, tree traversals (pre/in/post/level), BFS and DFS on graphs and
grids, 1D dynamic programming, bit tricks (XOR, power of two), simulation and matrix traversal.

**Intermediate**
Variable sliding window, prefix sum + hashmap, Kadane, Dutch national flag, intervals (merge,
insert, sweep), monotonic stack, monotonic deque, binary search on the answer, rotated-array and
peak binary search, fast and slow pointers, reverse and merge linked lists, LRU (doubly linked
list + map), tree recursion returning info, LCA, BST properties, heap top-K, K-way merge, two
heaps, backtracking (subsets, permutations, combinations), multi-source BFS, grid flood fill,
cycle detection, topological sort (DFS and Kahn), bipartite check, Union-Find, knapsack (0/1 and
unbounded), LIS, LCS and edit distance, grid DP, greedy (intervals, jump game), trie, cyclic sort,
Boyer-Moore voting, difference array, quickselect, `s+s` rotation trick, palindrome expand-around-center,
anagram window.

**Advanced**
Dijkstra, Bellman-Ford, Floyd-Warshall, minimum spanning tree (Kruskal, Prim), 0-1 BFS,
bidirectional BFS, Tarjan (SCC, bridges, articulation points), Hierholzer (Eulerian path),
interval DP, bitmask DP, state-machine DP, DP on trees, digit DP, segment tree, Fenwick tree,
sparse table, binary lifting, KMP and Z-function, rolling hash (Rabin-Karp), Manacher,
suffix structures, Aho-Corasick, sqrt decomposition and Mo's algorithm, heavy-light decomposition,
max flow and bipartite matching, meet in the middle, matrix exponentiation, reservoir sampling,
probabilistic structures (Bloom filter, HyperLogLog, count-min sketch).

Competitive-programming-only items (segment tree variants, suffix structures, flows, Mo's, HLD)
show up rarely in standard backend interviews. Learn them last.

---

## Part 2: signal to pattern, by area

### Arrays and strings
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Hashing / frequency count | "pair that sums to", "duplicates", "anagram", "count occurrences" | Remember what you've seen in a map so each lookup is O(1) | Two Sum, Group Anagrams | Chat | High |
| Two pointers (sorted or opposite ends) | "pair/triplet that sums to X", "palindrome", sorted input | Which pointer moves when the sum is too small or too big? Skip duplicates | 3Sum, Container With Most Water | Chat | High |
| Fast and slow pointers on arrays | "remove duplicates in place", "partition", "find the duplicate" | One pointer reads, one writes; or tortoise and hare on index values | Remove Duplicates, Move Zeroes | To learn | Medium |
| Sliding window (fixed) | "every window of size k", "max sum of k consecutive" | Add the new element, drop the old one | Max Average Subarray | Earlier | High |
| Sliding window (variable) | "longest/shortest substring or subarray with property" | What makes the window valid, and when do I shrink it? | Longest Substring Without Repeating | Earlier | High |
| Prefix sum | "sum of range i..j", many range queries | Precompute running totals so any range is one subtraction | Range Sum Query | Chat | High |
| Prefix sum + hashmap | "number of subarrays with sum k", "longest subarray with sum 0 or equal 0s and 1s" | Look up `prefix - k`. Count needs counts, longest needs first index. Seed the empty prefix | Subarray Sum = K, Contiguous Array | Chat | High |
| Difference array | "add a value to every element in range l..r" many times | Mark +v at l and -v at r+1, then take a prefix sum | Corporate Flight Bookings, Car Pooling | To learn | Low |
| Kadane | "maximum sum contiguous subarray" | Best sum ending here = max(element, element + previous best) | Maximum Subarray | Earlier | High |
| Dutch national flag | "sort 0/1/2", "partition around a pivot" | Three pointers: low, mid, high | Sort Colors | Earlier | Medium |
| Boyer-Moore voting | "element appearing more than n/2 times" | Cancel out pairs of different elements | Majority Element | To learn | Low |
| Cyclic sort / index as hash | "numbers in range 1..n", "find missing or duplicate" | Put each value at index value-1 | First Missing Positive | To learn | Medium |
| Intervals | "merge", "overlapping", "meeting rooms", "insert interval" | Sort by start, then compare with the last interval | Merge Intervals, Insert Interval | Earlier | High |
| Sweep line | "max number of overlapping intervals at once" | Turn each interval into +1 and -1 events, sort, walk | Meeting Rooms II | To learn | Medium |
| Monotonic stack | "next greater/smaller element", "histogram", "daily temperatures" | Keep the stack increasing or decreasing; pop while the new value breaks the order | Daily Temperatures, Largest Rectangle in Histogram | Earlier | High |
| Monotonic deque | "maximum in every sliding window" | Deque of indexes with decreasing values | Sliding Window Maximum | To learn | Medium |
| Sorting + custom comparator | "arrange to form the largest number", "sort by multiple keys" | What order makes the answer fall out of a sorted list? | Largest Number | To learn | Medium |
| Matrix traversal | "spiral", "rotate image", "set matrix zeroes" | Track four boundaries, or transpose then reverse | Spiral Matrix, Rotate Image | To learn | Medium |
| `s + s` rotation trick | "is B a rotation of A", "repeated substring pattern" | Every rotation of s is a substring of s+s | Rotate String, Repeated Substring Pattern | Chat | Medium |
| Palindrome: expand around center | "longest palindromic substring", "count palindromic substrings" | Try every center (odd and even) and expand | Longest Palindromic Substring | To learn | Medium |
| KMP / Z-function | "find a pattern in a long text in O(n+m)" | A precomputed table avoids rescanning matched characters | Find Index of First Occurrence | Chat (briefly) | Low |
| Rolling hash (Rabin-Karp) | "repeated substrings", "compare many substrings fast" | Hash a window and update it in O(1) | Longest Duplicate Substring | To learn | Low |
| Manacher | "longest palindrome in O(n)" | Reuse palindrome info across centers | Longest Palindromic Substring (optimal) | To learn | Low |

### Binary search family
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Standard binary search | sorted array, "find target" | Keep the half that can contain the answer | Binary Search | To learn | High |
| Lower and upper bound | "first or last position", "insert position" | Binary search that keeps going after a match | Find First and Last Position | To learn | High |
| Rotated sorted array | "sorted array rotated at a pivot" | One half is always sorted; decide which | Search in Rotated Sorted Array | To learn | High |
| Peak finding | "find a peak element" | Move toward the larger neighbor | Find Peak Element | To learn | Medium |
| Binary search on the answer | "minimum capacity / speed / time such that ...", "minimize the maximum" | Is "feasible for X" monotonic? Then search X, with a check function | Koko Eating Bananas, Capacity to Ship Packages, Split Array Largest Sum | To learn | High |
| Binary search on a 2D matrix / on value | "k-th smallest in a sorted matrix" | Search the value range and count how many are `<=` mid | Kth Smallest in Sorted Matrix | To learn | Medium |

### Linked list
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Reverse a list | "reverse", "reverse in groups" | Three pointers: prev, curr, next | Reverse Linked List | Chat 2 | High |
| Fast and slow pointers | "cycle", "middle", "is palindrome" | Slow moves 1, fast moves 2 | Linked List Cycle, Middle of the Linked List | Chat 2 (middle only) | High |
| Dummy head node | "remove nodes", "merge" | A dummy node removes the head special case | Merge Two Sorted Lists, Remove Nth Node | To learn | High |
| Gap pointers | "n-th from the end" | Move one pointer n steps ahead first | Remove Nth Node From End | To learn | Medium |
| Find middle + reverse + merge | "reorder list" | Combine three small techniques | Reorder List | To learn | Medium |
| Doubly linked list + map | "LRU cache" | Map gives O(1) lookup, list gives O(1) reorder | LRU Cache | To learn | High |
| Copy with a node map | "deep copy with random pointers" | Map old node to new node, then link | Copy List with Random Pointer | To learn | Medium |

### Stack and queue
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Stack for matching | "valid parentheses", "nested structure", "decode string" | Push on open, pop and compare on close | Valid Parentheses, Decode String | To learn | High |
| Expression evaluation | "calculator", "postfix", "infix" | Operand stack plus operator stack | Basic Calculator, Evaluate RPN | To learn | Medium |
| Min stack / design stacks | "get min in O(1)" | Store the running min alongside each value | Min Stack | To learn | Medium |
| Queue using stacks (and reverse) | "implement a queue with two stacks" | Move between stacks lazily | Implement Queue using Stacks | To learn | Low |
| Circular queue / ring buffer | "fixed-size buffer", "hit counter" | Head and tail indexes modulo size | Design Circular Queue | To learn | Low |

### Trees
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Tree DFS traversals | "preorder / inorder / postorder" | Where do I do the work: before, between or after the children? | Binary Tree Inorder Traversal | Earlier | High |
| Level-order BFS | "level by level", "right side view", "zigzag" | Queue with a size snapshot per level | Binary Tree Level Order Traversal | Chat (BFS levels) | High |
| Recursion returning info | "height", "diameter", "balanced", "max path sum" | Each call returns a value up; update a global answer on the way | Diameter of Binary Tree, Maximum Path Sum | Earlier + Chat 2 | High |
| Lowest common ancestor | "common ancestor of two nodes" | Return the node when both sides report a hit | Lowest Common Ancestor | Earlier | High |
| BST properties | "validate BST", "k-th smallest", "inorder successor" | Inorder traversal of a BST is sorted; carry min and max bounds | Validate BST, Kth Smallest in BST | Earlier + Chat 2 | High |
| Construct a tree from traversals | "build tree from preorder and inorder" | Preorder gives the root; inorder splits left and right | Construct Binary Tree from Preorder and Inorder | To learn | Medium |
| Serialize and deserialize | "encode a tree to a string and back" | Preorder with null markers, or level order | Serialize and Deserialize Binary Tree | To learn | Medium |
| Tree DP | "rob houses on a tree", "max with constraints on parent/child" | Return a pair of states from each node | House Robber III | To learn | Medium |
| Ancestor walk with parent pointers | "is C an ancestor of B" in a parent-array tree | Walk up from B toward the root | Reachability in a town tree | Chat (discussed) | Low |
| Binary lifting | "k-th ancestor", fast LCA with many queries | Precompute 2^j-th ancestors | Kth Ancestor of a Tree Node | To learn | Low |
| Morris traversal | "inorder in O(1) space" | Temporary threads to the predecessor | Inorder Traversal (follow-up) | To learn | Low |
| Segment tree | "range sum/min with point updates" | Tree of ranges, O(log n) per query and update | Range Sum Query Mutable | To learn | Low |
| Fenwick tree (BIT) | "prefix sums with updates", "count of smaller elements after self" | Compact segment-tree alternative for prefix queries | Range Sum Query Mutable | To learn | Low |
| Sparse table | "range minimum, no updates" | Precompute min over power-of-two ranges | Range Minimum Query | To learn | Low |

### Heap and priority queue
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Top K / K-th largest | "top K frequent", "K closest", "K-th largest" | Min-heap of size K, or quickselect | Top K Frequent Elements, Kth Largest Element | To learn | High |
| K-way merge | "merge K sorted lists or streams" | Heap holds the current head of each list | Merge K Sorted Lists | To learn | High |
| Two heaps | "median of a stream", "sliding median" | Max-heap for the lower half, min-heap for the upper half | Find Median from Data Stream | To learn | Medium |
| Scheduling with a heap | "task scheduler", "meeting rooms", "minimum rooms" | Heap of end times or of remaining counts | Task Scheduler, Meeting Rooms II | To learn | Medium |
| Quickselect | "k-th element in O(n) average" | Partition like quicksort, recurse into one side | Kth Largest Element | To learn | Medium |
| Heap with lazy deletion | "remove arbitrary elements from a heap" | Mark as removed and skip when popped | Sliding Window Median | To learn | Low |

### Graphs
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| BFS on a graph | "can you reach", "is there a path", "connected" | Queue and visited, mark at push. Directed or undirected? | Find if Path Exists in Graph | Chat | High |
| Connected components | "number of groups/provinces/islands" | Loop over all nodes; search only from unvisited | Number of Provinces, Number of Islands | Chat | High |
| BFS shortest path (unweighted) | "minimum steps/moves/minutes", equal cost moves | Count BFS levels | Shortest Path in Binary Matrix, Open the Lock | Chat | High |
| Multi-source BFS | "spread from many sources at once", "distance to nearest X" | Seed the queue with every source | Rotting Oranges, 01 Matrix | Chat | High |
| Grid DFS / flood fill | "count regions", "area of the largest region" | Mark visited before recursing; base case covers bounds and not-land | Max Area of Island, Closed Islands | Chat | High |
| Two-pass flood fill | "enclosed", "surrounded", "touches the border" | Remove the border-connected cells first, then count the rest | Closed Islands, Surrounded Regions | Chat | Medium |
| Reverse search from targets | "can reach both X and Y" | Search from the destinations backward and intersect | Pacific Atlantic Water Flow | Chat | Medium |
| Bipartite check (2-coloring) | "split into two groups with no conflict" | Color neighbors opposite, conflict means no | Is Graph Bipartite, Possible Bipartition | Chat | Medium |
| Cycle detection, directed | "can all tasks finish", "is there a cycle" | DFS with visited vs in-progress sets, or Kahn's | Course Schedule | Chat | High |
| Cycle detection, undirected | "is this graph a tree", "redundant connection" | DFS with a parent check, or Union-Find | Redundant Connection, Graph Valid Tree | To learn | Medium |
| Topological sort | "ordering with dependencies" | DFS finish order, or Kahn's with indegree | Course Schedule II, Alien Dictionary | Chat | High |
| Union-Find (DSU) | "are X and Y connected", groups merge over time, "accounts merge" | find() with path compression, union() by rank | Number of Provinces, Accounts Merge | To learn | High |
| Dijkstra | "shortest path with positive weights", "minimum cost/effort/time" | Min-heap of (distance, node) | Network Delay Time, Path With Minimum Effort | To learn | High |
| Bellman-Ford | "shortest path with negative edges", "at most K stops" | Relax all edges up to V-1 (or K+1) times | Cheapest Flights Within K Stops | To learn | Low |
| Floyd-Warshall | "shortest paths between all pairs", small n | Try each node as an intermediate | Find the City With the Smallest Number of Neighbors | To learn | Low |
| Minimum spanning tree | "connect all points at minimum cost" | Kruskal (sort edges + DSU) or Prim (heap) | Min Cost to Connect All Points | To learn | Medium |
| 0-1 BFS | edges with weight 0 or 1 | Deque: push weight-0 neighbors to the front | Minimum Obstacle Removal | To learn | Low |
| Bidirectional BFS | "shortest transformation sequence", huge search space | Search from both ends and meet in the middle | Word Ladder | To learn | Low |
| Implicit graph | "word changes one letter at a time", puzzle states, lock combinations | Each state is a node, each legal move is an edge | Word Ladder, Open the Lock | To learn | Medium |
| Clone a graph | "deep copy a graph" | Map old node to new node while traversing | Clone Graph | To learn | Medium |
| Tarjan / Kosaraju | "strongly connected components", "critical connections", "bridges" | One DFS with discovery and low-link times | Critical Connections in a Network | To learn | Low |
| Eulerian path (Hierholzer) | "use every edge exactly once", "reconstruct itinerary" | Post-order DFS on edges, then reverse | Reconstruct Itinerary | To learn | Low |
| Max flow / bipartite matching | "assign workers to jobs", "maximum matching" | Augmenting paths | (rare in interviews) | To learn | Low |

### Backtracking and recursion
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Subsets / combinations | "all subsets", "all combinations that sum to X" | Include or exclude; undo after recursing | Subsets, Combination Sum | Chat (similar) + Chat 2 | High |
| Permutations | "all arrangements / orderings" | Track used elements; undo | Permutations | Chat 2 | High |
| Choice per position | "all letter combinations", "generate parentheses" | One level per position; append, recurse, delete | Letter Combinations of a Phone Number | Chat + Chat 2 | High |
| Handling duplicates | "unique results from input with duplicates" | Sort, then skip equal values at the same level | Subsets II, Combination Sum II | Chat 2 | Medium |
| Constraint satisfaction | "place N queens", "solve sudoku", "word search" | Place, check validity, recurse, undo; prune early | N-Queens, Word Search | Chat 2 | Medium |
| Partitioning | "split into palindromes" | Try every cut point | Palindrome Partitioning | To learn | Medium |
| Divide and conquer | "split, solve halves, combine" | Merge sort, count inversions, closest pair | Merge Sort, Count of Smaller Numbers | To learn | Medium |
| Meet in the middle | n up to about 40 and brute force is 2^n | Split in two halves, combine with a map | Closest Subsequence Sum | To learn | Low |

### Dynamic programming
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| 1D DP | "ways to climb", "max you can rob", "decode ways" | State = position. Recurrence from the previous one or two | Climbing Stairs, House Robber, Decode Ways | Earlier + Chat 2 | High |
| 0/1 knapsack / subset sum | "can you pick items to hit exactly X", "partition equal subset" | State = (item, remaining capacity) | Partition Equal Subset Sum, Target Sum | Earlier + Chat 2 | High |
| Unbounded knapsack | "coins, unlimited use", "minimum coins" | Same item can be used again | Coin Change, Word Break | Chat 2 | High |
| LIS | "longest increasing subsequence" | O(n²) DP, or O(n log n) with binary search | Longest Increasing Subsequence | Earlier + Chat 2 | Medium |
| LCS / edit distance | "two strings", "minimum edits", "longest common subsequence" | State = (i, j) prefixes | Longest Common Subsequence, Edit Distance | Earlier + Chat 2 | High |
| Grid DP | "unique paths", "minimum path sum" | State = cell. Come from top or left | Unique Paths, Minimum Path Sum | To learn | High |
| Palindrome DP | "longest palindromic subsequence", "count palindromic substrings" | State = (i, j) substring | Longest Palindromic Subsequence | To learn | Medium |
| State-machine DP | "stock with cooldown", "at most K transactions" | State = holding or not holding, plus extras | Best Time to Buy and Sell Stock with Cooldown | To learn | Medium |
| Interval DP | "burst balloons", "matrix chain", "merge stones" | State = (left, right) range | Burst Balloons | To learn | Low |
| Bitmask DP | "visit all nodes", n about 15 to 20 | State = (visited-mask, last node) | Shortest Path Visiting All Nodes | To learn | Low |
| DP on trees | "max with parent/child constraints" | Return states from each child | House Robber III | To learn | Medium |
| Digit DP | "count numbers up to N with a digit property" | State = (position, tight, extra) | Numbers With Repeated Digits | To learn | Low |
| Memoization vs tabulation, space optimization | any DP | Top-down first, then bottom-up, then keep only the rows you need | (all) | Earlier + Chat 2 | High |

### Greedy
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| Interval scheduling | "maximum non-overlapping", "minimum removals" | Sort by end time, take what fits | Non-overlapping Intervals | Earlier + Chat 2 | High |
| Jump game / reachability | "can you reach the end", "minimum jumps" | Track the furthest reach | Jump Game, Jump Game II | To learn | Medium |
| Sort + greedy choice | "assign cookies", "minimum arrows" | Sort, then pair or take the locally best | Assign Cookies | To learn | Medium |
| Greedy + heap | "task scheduling", "reorganize string" | Always take the largest remaining | Task Scheduler, Reorganize String | To learn | Medium |
| Gas station / circular | "can you complete the circuit" | If the running sum drops below 0, restart after that point | Gas Station | To learn | Low |
| Huffman / fractional knapsack | "optimal prefix code" | Merge the two smallest with a heap | (classic) | To learn | Low |

### Bit manipulation and math
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| XOR tricks | "every number appears twice except one" | a ^ a = 0 and a ^ 0 = a | Single Number | To learn | Medium |
| Count set bits | "number of 1 bits" | n & (n-1) clears the lowest set bit | Number of 1 Bits, Counting Bits | To learn | Medium |
| Power of two / bit checks | "is power of two", "get/set/clear bit" | n & (n-1) == 0 | Power of Two | To learn | Medium |
| Bitmask subsets | "all subsets with n <= 20" | Each mask from 0 to 2^n - 1 is one subset | Subsets (bitmask) | To learn | Low |
| GCD / LCM | "common divisor", "simplify fraction" | Euclid's algorithm | Greatest Common Divisor of Strings | To learn | Medium |
| Sieve of Eratosthenes | "count primes below n" | Cross out multiples | Count Primes | To learn | Medium |
| Fast exponentiation / modular arithmetic | "x to the power n", "answer modulo 1e9+7" | Square and halve the exponent | Pow(x, n) | To learn | Medium |
| Combinatorics (nCr mod p) | "number of ways to arrange / paths" | Factorials with modular inverse | Unique Paths (math) | To learn | Low |
| Matrix exponentiation | "n-th Fibonacci for huge n" | Raise a transition matrix to a power | (rare) | To learn | Low |

### Design-style data structures (common in backend interviews)
| Pattern | Signal in the question | Core idea / first question | Example problems | Status | Priority |
|---|---|---|---|---|---|
| LRU / LFU cache | "evict least recently or frequently used" | Hash map + linked list (LRU), plus frequency buckets (LFU) | LRU Cache, LFU Cache | To learn | High |
| Randomized set | "insert, delete, get random in O(1)" | Array + map of value to index; swap with the last on delete | Insert Delete GetRandom O(1) | To learn | Medium |
| Time-based key-value store | "value at or before time t" | Map of key to a list sorted by time, then binary search | Time Based Key-Value Store | To learn | Medium |
| Trie | "autocomplete", "words with a common prefix", "word search II" | Node per character with children and an end flag | Implement Trie, Word Search II | To learn | High |
| Rate limiter / hit counter | "requests in the last 5 minutes" | Queue of timestamps, or sliding window counters | Design Hit Counter | To learn | Medium |
| Merge feeds | "news feed / timeline from many users" | K-way merge with a heap | Design Twitter | To learn | Low |
| Reservoir sampling | "pick a random element from a stream of unknown length" | Keep the i-th item with probability 1/i | Linked List Random Node | To learn | Low |
| Probabilistic structures | "membership at huge scale", "count distinct", "frequency estimates" | Bloom filter, HyperLogLog, count-min sketch (more of a system design topic) | (system design) | To learn | Low |

---

## Part 3: decision aids

### Constraints tell you the target complexity
| Input size | Target | Usually means |
|---|---|---|
| n <= 20 | O(2^n), O(n!) | Backtracking, subsets, permutations, bitmask |
| n <= 40 | O(2^(n/2)) | Meet in the middle |
| n <= 100 | O(n^3) | Triple loops, small DP, Floyd-Warshall |
| n <= 1,000 to 5,000 | O(n^2) | Nested loops, 2D DP, grid problems |
| n <= 100,000 | O(n log n) or O(n) | Sorting, heap, binary search, two pointers, sliding window, hashing, BFS/DFS |
| n >= 1,000,000 | O(n) or O(log n) | One pass, math, binary search on a formula |

### What is the question asking for?
| Asked for | Look at |
|---|---|
| "Does it exist / is it possible" | BFS/DFS, cycle detection, DP (boolean), greedy |
| "How many ways / how many" | DP counting, backtracking count, hashmap counts (prefix sum) |
| "Minimum / maximum value" | DP, greedy, binary search on the answer, heap |
| "Shortest path / minimum steps" | BFS (equal cost), Dijkstra (weights), 0-1 BFS |
| "Longest / shortest subarray or substring" | Sliding window, prefix sum + first index |
| "All possibilities" | Backtracking |
| "Top K / K-th" | Heap, quickselect, binary search on value |
| "Order / sequence respecting rules" | Topological sort |
| "Groups / connected" | Union-Find, DFS/BFS components |

### Greedy, DP or backtracking?
1. Can a local best choice never hurt later (you can argue it)? Try **greedy** and test it on a counterexample.
2. Do subproblems repeat with the same inputs? Use **DP** (state, choices, base case).
3. Do you need to list every solution, or is n tiny? Use **backtracking**.
4. If greedy fails on a small counterexample, it's usually DP.

### Graphs hide in many costumes
You may never see the word "graph". Treat it as a graph when you see:
- **A grid:** each cell is a node and its 4 neighbors are the edges.
- **A list of pairs:** `[a, b]` meaning "a needs b", "a and b are friends", "a connects to b".
- **States that change by one move:** a lock combination, a word changing one letter at a time, a puzzle board.
- **Dependencies or ordering rules:** that points to a directed graph and topological sort.

### BFS or DFS?
| Use BFS when... | Use DFS when... |
|---|---|
| You need the **shortest** path or minimum steps (equal cost per move) | You need to explore **everything** in a region (count, size, flood fill) |
| Things spread from **many sources at once** | You need cycle detection or finish order |
| Recursion depth could overflow the stack | The recursive structure is natural (trees, backtracking) |

### Traps to check before you code
- **Direction:** undirected means add both directions. Directed means decide the direction from what the algorithm needs.
- **Visited timing:** BFS marks at push. DFS marks before recursing.
- **Labels:** are nodes numbered from 0 or from 1? That changes your array size.
- **Disconnected input:** loop over every node and start a search only from unvisited ones.
- **Empty or degenerate input:** `[]`, `""`, one cell, `source == destination`.
- **Count or longest?** A map storing counts and one storing first indexes solve different questions.
- **Off-by-one on bounds:** valid indexes are `0` to `length - 1`.
- **Complexity:** state it yourself, naming V and E, or m and n. Remember the cost of a library sort.

### If nothing matches
1. Solve a tiny example by hand and watch what you do. That is usually the algorithm.
2. Write the brute force and find what it recomputes.
3. Ask: can I sort it, hash it, search the answer space, or use a heap?
4. Say your reasoning out loud. Interviewers score the thinking, not only the final code.

---

## Part 4: suggested study order from here (my suggestion)
1. **Heaps and top-K** (high priority, not yet covered)
2. **Binary search family**, especially search on the answer
3. **Union-Find** and **Dijkstra** (finish the graph toolkit)
4. **Trie** and **LRU cache** (common in backend interviews)
5. **Linked list** patterns and **tree** patterns (LCA, BST)
6. **DP refresh**, then unbounded knapsack and grid DP
7. Everything marked Low only if time remains
