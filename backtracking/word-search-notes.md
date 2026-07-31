# Word Search — Notes

> **Problem:** Given a 2D grid of characters and a word, return true if the word exists in the grid formed by sequentially adjacent cells (up, down, left, right). Same cell can't be used twice.
> LC 79 — leetcode.com/problems/word-search

---

## The pattern

Word Search = backtracking on a 2D grid where you **match characters one by one**, marking cells visited as you go.

- Outer loop finds every possible starting cell
- DFS matches characters moving in 4 directions
- Mark cell visited on choose, restore on unchoose
- Return `true` the moment full word is matched

---

## Solution

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        for (int i = 0; i < board.length; i++)
            for (int j = 0; j < board[0].length; j++)
                if (function(i, j, board, word, 0)) return true;
        return false;
    }

    private boolean function(int row, int col, char[][] board, String word, int index) {
        // bounds first — always
        if (row < 0 || row >= board.length || col < 0 || col >= board[0].length) return false;
        // visited and character check
        if (board[row][col] == '#' || board[row][col] != word.charAt(index)) return false;
        // base case — last character matched
        if (index == word.length() - 1) return true;

        char temp = board[row][col];
        board[row][col] = '#';                           // CHOOSE — mark visited

        boolean found = function(row+1, col, board, word, index+1)
                     || function(row-1, col, board, word, index+1)
                     || function(row, col+1, board, word, index+1)
                     || function(row, col-1, board, word, index+1);

        board[row][col] = temp;                          // UNCHOOSE — always restore

        return found;
    }
}
```

---

## My mistakes and why they happened

### Bug 1 — Fixed starting cell `(0,0)`

```java
return function(0, 0, board, word, 0);  // ❌ only tries top-left corner
```

**Why it happened:** every problem before this had one clear entry point. Word Search is the first problem where the entry point itself needs to be searched — a new structural idea.

**Fix:** outer loop over all cells:
```java
for (int i = 0; i < board.length; i++)
    for (int j = 0; j < board[0].length; j++)
        if (function(i, j, board, word, 0)) return true;
return false;
```

**Signal:** ask "where can the answer START?" If anywhere → outer loop before recursion.

---

### Bug 2 — Base case before character check

```java
if (index == word.length()-1) return true;              // ❌ success before verifying last char
if (board[row][col] != word.charAt(index)) return false;
```

**Why it happened:** in every previous problem, reaching the right index automatically meant success. Here reaching the right index isn't enough — the value must also match.

**Fix:** character check before base case:
```java
if (board[row][col] != word.charAt(index)) return false; // verify first
if (index == word.length()-1) return true;               // then declare success
```

**Signal:** ask "does reaching the base case guarantee success, or do I still need to verify something?"

---

### Bug 3 — Bounds check after array access

```java
if (word.charAt(index) != board[row][col]   // ❌ accesses board before checking bounds
    || row >= board.length || row < 0...)
```

**Why it happened:** wrote the most important condition (character match) first. But Java evaluates left to right — `board[row][col]` crashes before bounds are checked.

**Fix — always in this order:**
```
bounds → visited → value
never:
value → bounds
```

```java
if (row < 0 || row >= board.length || col < 0 || col >= board[0].length) return false;
if (board[row][col] == '#' || board[row][col] != word.charAt(index)) return false;
```

---

### Bug 4 — Ignoring return values from recursive calls

```java
function(row+1, col, board, word, index+1);  // ❌ return value thrown away
function(row-1, col, board, word, index+1);
function(row, col+1, board, word, index+1);
function(row, col-1, board, word, index+1);
board[row][col] = temp;
return false;                                // always returns false!
```

**Why it happened:** every previous backtracking problem was `void` — collected into a list, never returned a value. Word Search is the first boolean backtracking problem. Muscle memory said "call the function, it handles itself."

**Fix:** propagate `true` upward with `||`:
```java
boolean found = function(row+1, col, board, word, index+1)
             || function(row-1, col, board, word, index+1)
             || function(row, col+1, board, word, index+1)
             || function(row, col-1, board, word, index+1);
board[row][col] = temp;   // unchoose always runs before return
return found;
```

`||` short-circuits — if any direction returns `true`, remaining directions don't run. Unchoose always happens before returning regardless.

---

## The visited trick — no extra space

Instead of a separate `boolean[][] visited` matrix:

```java
char temp = board[row][col];
board[row][col] = '#';     // CHOOSE — mark visited in-place
dfs(...);                  // EXPLORE
board[row][col] = temp;    // UNCHOOSE — restore automatically
```

Clever and clean — no extra space, perfect symmetric unchoose built in.

---

## Find-all vs find-any — the pattern that changed

| Goal | Return type | Pattern |
|---|---|---|
| Find ALL solutions | `void` | collect into list, ignore return values |
| Find IF any solution exists | `boolean` | propagate `true` upward with `\|\|` |

Subsets, permutations, combinations, N-Queens → all `void`, collect into list.
Word Search → `boolean`, stop and return the moment one solution is found.

---

## The 4-question checklist — ask before coding any grid problem

```
1. Where can the answer START?
   → one fixed point → single entry call
   → anywhere in grid → outer loop over all cells

2. Does reaching the base case guarantee success,
   or do I still need to verify something?
   → automatic → base case alone is enough
   → needs verification → check value BEFORE declaring success

3. Am I checking bounds BEFORE accessing the array?
   → always: bounds → visited → value
   → never: value → bounds

4. Is this find-all or find-any?
   → find-all → void, collect into list
   → find-any → boolean, propagate true with ||
```

---

## How Word Search builds on grid paths

| | Grid paths | Word Search |
|---|---|---|
| Directions | right, down (2) | all 4 |
| Stop — success | reached destination | matched all characters |
| Stop — fail | out of bounds | out of bounds, wrong char, visited |
| Visited tracking | not needed | in-place `'#'` marking |
| Returns | void | boolean |
| Entry point | always `(0,0)` | any cell — outer loop |
