# My Personal Problem-Solving Instructions

> Built from my actual mistakes across recursion, backtracking, linked lists, and DP.
> Read this BEFORE starting any new problem. Not after getting stuck.

---

## The Universal 3-Step Process (apply to EVERY problem)

```
Step 1 — IDENTIFY the pattern (which category is this?)
Step 2 — ANSWER the checklist for that category (before writing any code)
Step 3 — WRITE code in order: recursion → memoization → tabulation
```

Never jump to code before completing Steps 1 and 2.

---

## Step 1 — Identify the Pattern

Ask these questions in order:

```
Q: Do I need ALL solutions (every subset, every permutation, every path)?
→ YES → BACKTRACKING

Q: Do I need ONE answer (minimum, maximum, count, true/false)?
→ YES → DP (recursion + memo)
   → Do the same subproblems repeat?
      YES → add memo
      NO  → plain recursion

Q: Am I computing a value on a fixed structure (tree, linked list)?
→ YES → PLAIN RECURSION (no loop, no memo usually)

Q: Am I traversing a grid looking for a path or word?
→ YES → GRID BACKTRACKING (choose/explore/unchoose + visited marking)
```

---

## Step 2A — Checklist for RECURSION problems

Answer these before writing any code:

```
□ What is the base case? (what input is trivially obvious?)
□ How do I shrink the problem? (what is the smaller version?)
□ How do I combine results? (what do I do with what came back?)
□ What does the function RETURN? (a value? void? boolean?)
□ Is there a base case for BOTH success AND failure?
  (arrived → return 1 or 0, overshot/invalid → return 0 or -1)
```

### My recurring mistakes in recursion:

**Mistake R1 — Missing base case or wrong value**
```
n==0 in climbing stairs → return 1 NOT 0
"arrived at goal" → return 1 (one valid path)
"overshot/invalid" → return 0 or -1
```

**Mistake R2 — Not using return values from recursive calls**
```java
function(node.left);   // ❌ return value ignored
if (function(node.left)) return true;  // ✅ propagate result
```

**Mistake R3 — Base case triggers before goal check**
```java
if (row >= m || col >= n) return;   // ❌ fires before destination check
if (row == m-1 && col == n-1) ...  // ← never reached

// ✅ ALWAYS: destination check BEFORE bounds check
if (row == m-1 && col == n-1) { record; return; }
if (row >= m || col >= n) return;
```

**Mistake R4 — Bounds check AFTER array access**
```java
if (board[row][col] != word.charAt(i) || row < 0 || ...)  // ❌ crashes
if (row < 0 || row >= m || col < 0 || col >= n) return;   // ✅ bounds FIRST
if (board[row][col] != word.charAt(i)) return;             // ✅ value SECOND
```

---

## Step 2B — Checklist for BACKTRACKING problems

Answer these before writing any code:

```
□ What is the base case? (when is one answer complete?)
□ What shared mutable state am I modifying? (list, board, string)
□ What is my CHOOSE step?
□ What is my UNCHOOSE step? (must be exact mirror of choose)
□ Do I need to SKIP duplicates? (if input has duplicates → sort first)
□ Find-all (void) or find-any (boolean + return true)?
□ Where can the answer START? (one point → single call, anywhere → outer loop)
```

### My recurring mistakes in backtracking:

**Mistake B1 — Missing unchoose**
```
Every choose has EXACTLY ONE matching unchoose after explore.
If you chose inside an if block → unchoose inside same if block.
If you chose before the loop → unchoose after the loop.
```

**Mistake B2 — Void when boolean needed**
```
Find ALL solutions → void, collect into list
Find ANY one solution → boolean, return true immediately on success
Sudoku, Word Search → boolean (find any)
Subsets, Permutations → void (find all)
```

**Mistake B3 — Fixed starting cell**
```
Word Search, LIS → answer can START anywhere → outer loop
Subsets, Permutations → always start from beginning → no outer loop
Signal: "does the problem say starting position matters?"
```

**Mistake B4 — Wrong pruning condition**
```
Subsets II:      if (i > start && nums[i] == nums[i-1]) continue
Permutations II: if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue

i > start  → skip only at same recursive level
i > 0 + !used[i-1] → skip only if previous copy already backtracked
```

**Mistake B5 — Adding index when not needed**
```
Combinations: index needed (can't go back)
Subsets:      index needed (can't go back)
Permutations: NO index (can use any unused element)
Coin Change:  NO index (can reuse any coin unlimited times)
Signal: "can I reuse choices?" → no index needed
```

**Mistake B6 — Wrong start index in combinations**
```java
function(n, k, value+1, ...);  // ❌ passes where THIS call started
function(n, k, i+1, ...);      // ✅ passes where THIS ITERATION ended
```
`i+1` not `start+1` — next call needs to know where YOUR decision left off.

---

## Step 2C — Checklist for LINKED LIST recursion problems

Answer these four questions before writing any code:

```
□ What does this function RETURN?
  → "head of correctly [verb]ed list starting from this node"

□ What is the base case?
  → null (nothing here) → return null
  → single node needs special handling → return head

□ What do I do with the recursion result?
  → wire to head.next (most problems)
  → use as newHead (reverse only)

□ What do I return?
  → head        (keep this node)
  → head.next   (remove this node)
  → newHead     (reverse — floats up from base case)
```

### My recurring mistakes in linked lists:

**Mistake L1 — Not wiring recursion result to head.next**
```java
ListNode temp = solve(head.next);   // ❌ head disconnected from fixed sublist
head.next = solve(head.next);       // ✅ wire result back to head
```

**Mistake L2 — else catches recursive case**
```java
if (l1 == null) return l2;
else if (l2 == null) return l1;
else return l1;   // ❌ catches EVERYTHING — recursive case never runs

if (l1 == null) return l2;  // ✅ narrow base cases only
if (l2 == null) return l1;  // ✅ recursive case falls through naturally
```

**Mistake L3 — Wrong base case for reverse**
```java
if (head == null || head.next == null) return null;  // ❌ loses the node
if (head == null || head.next == null) return head;  // ✅ single node IS new head
```

**Mistake L4 — Returning wrong pointer**
```java
return head;      // keep this node
return head.next; // remove this node (skip it)
return newHead;   // reverse — new head from base case
```

---

## Step 2D — Checklist for DP problems

Answer these FIVE questions before writing any code:

```
□ 1. What changes between recursive calls?
     → that is your memo KEY and array SIZE

□ 2. Is the answer the same as the return value?
     → YES → track inside function, return it directly
     → NO  → external variable (like maxDiameter, or outer loop like LIS)

□ 3. How many choices at each step?
     → fixed (2-3) → write each call explicitly
     → variable    → loop over choices

□ 4. What does the base case mean?
     → arrived at goal exactly → return 1 or 0 (cost)
     → overshot/invalid        → return 0 or -1

□ 5. Fill direction for tabulation?
     → f(i - something) → needs LEFT  → fill LEFT  TO RIGHT →→→
     → f(i + something) → needs RIGHT → fill RIGHT TO LEFT  ←←←
```

### My recurring mistakes in DP:

**Mistake D1 — Wrong memo size**
```java
memo = new int[coins.length+1];  // ❌ sized by coins not by amount
memo = new int[amount+1];        // ✅ sized by what CHANGES between calls
```
Rule: memo size = range of values the changing parameter can take.

**Mistake D2 — Wrong sentinel for memo**
```
-1 is a valid answer in coin change (means impossible)
→ use -2 as sentinel
Always pick a value that can NEVER be a real answer.
```

**Mistake D3 — Adding index when not needed**
```
Combinations: index needed (restricted choices)
Coin change:  NO index (all coins always available, unlimited reuse)
Signal: "does the set of available choices change between calls?"
  YES → pass index
  NO  → no index
```

**Mistake D4 — Storing in wrong memo slot**
```java
// inside solve(index), inside a loop over j:
memo[j] = ...;      // ❌ storing in HELPER's slot
memo[index] = ...;  // ✅ storing in SUBJECT's slot

// Rule: inside solve(X), ONLY write to memo[X]
// j is just a tool — never store in memo[j] from inside solve(index)
```

**Mistake D5 — Returning dp[n-1] instead of global max**
```
Some problems: solve(0) gives the full answer → return dp[0] or dp[n-1]
LIS: solve(i) gives LIS ending at i → answer is MAX across all i
Signal: "does calling solve on one specific input give the full answer?"
  YES → return that one dp value
  NO  → track max/min across all positions with outer loop
```

**Mistake D6 — Wrong prefill value for dp array**
```
Prefill = base case value (what dp[i] equals before any j helps it)
LIS base case = 1 (element alone) → Arrays.fill(dp, 1)
Coin change   = impossible        → Arrays.fill(dp, amount+1)
Climbing stairs = explicit dp[0]=1, dp[1]=1
```

**Mistake D7 — Wrong loop order in tabulation**
```java
// Coin change — wrong
for (int i = 0; i < coins.length; i++)   // ❌ outer loop over coins
    for (int j = 1; j <= amount; j++)

// Coin change — correct
for (int i = 1; i <= amount; i++)         // ✅ outer = recursive argument
    for (int coin : coins)                 // ✅ inner = loop inside recursion
```
Rule: outer loop = recursive function's argument. Inner loop = loop inside recursion.

**Mistake D8 — Integer.MAX_VALUE overflow in tabulation**
```java
Arrays.fill(dp, Integer.MAX_VALUE);  // ❌ 1 + MAX_VALUE overflows
Arrays.fill(dp, amount+1);           // ✅ safely larger than any valid answer
```

---

## Step 3 — Code Writing Order

Always write in this order:

```
1. Plain recursion first (no memo)
   → get the logic right, verify base cases

2. Add memoization
   → three additions only:
      a. create memo array, fill with sentinel
      b. check memo at TOP of function (before any computation)
      c. store result in memo BEFORE returning

3. Convert to tabulation (if needed)
   → outer loop = recursive function's argument
   → inner loop = loop inside recursive function
   → prefill with base case value
   → fill in correct direction
   → return correct cell (single value or global max)
```

---

## The Subject/Helper/Accumulator Mental Model

This applies to EVERY problem with a loop inside recursion (LIS, Coin Change):

```
SUBJECT     = who we're computing for = the function's argument
HELPER      = who we ask for information = loop variable j
ACCUMULATOR = collects the best answer for SUBJECT = local variable (best, min, max)

Inside solve(SUBJECT):
  ACCUMULATOR starts at base value (1, 0, MAX, etc.)
  Loop over HELPERs:
    Ask HELPER for its answer: solve(j) or dp[j]
    Update ACCUMULATOR: best = max(best, 1 + solve(j))
  Store ACCUMULATOR in memo[SUBJECT]
  Return memo[SUBJECT]

NEVER store anything in memo[HELPER] from inside solve(SUBJECT).
```

---

## Quick Reference — When to use what

### Tracking variable

```
answer == return value   → track INSIDE function (coin change min, LIS best)
answer != return value   → track OUTSIDE function (diameter maxDiameter, subsets result list)
```

### Loop inside recursion

```
fixed choices (2-3)   → write each call explicitly
variable choices      → loop over choices, recurse inside
```

### Memo dimensions

```
1 changing parameter  → 1D memo
2 changing parameters → 2D memo
```

### Return value in linked lists

```
keep node    → return head
remove node  → return head.next
reverse      → return newHead (floats up from base case)
```

### Backtracking — find-all vs find-any

```
find ALL → void, collect into list, ignore return values
find ANY → boolean, return true immediately, skip unchoose on success
```

---

## Before Every Problem — The 60-Second Checklist

```
1. Which category? (recursion / backtracking / linked list / DP)
2. What does the function return?
3. What is the base case? (both success AND failure)
4. What changes between calls? (memo key)
5. Fixed or variable choices? (explicit calls or loop)
6. Find-all or find-any? (void or boolean)
7. Where can answer start? (single call or outer loop)
8. Fill direction? (for DP tabulation)
```

Write answers on paper before typing a single line of code.
If you can't answer all 8, you're not ready to code yet — re-read the problem.
