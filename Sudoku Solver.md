# Sudoku Solver — Notes

> **Problem:** Fill a 9×9 partially filled Sudoku board so every row, column, and 3×3 box contains digits 1–9 exactly once.
> LC 37 — leetcode.com/problems/sudoku-solver

---

## The pattern

Sudoku = backtracking where you **find the next empty cell, try digits 1–9, recurse, backtrack**.

Same skeleton as N-Queens — place one value, check constraints, recurse, undo. The only new idea is the box constraint.

---

## Solution

```java
class Solution {
    public void solveSudoku(char[][] board) {
        boolean[][] rows = new boolean[9][10];
        boolean[][] cols = new boolean[9][10];
        boolean[][] boxes = new boolean[9][10];

        // pre-fill constraints from existing digits — once, before recursion
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] != '.') {
                    int num = board[i][j] - '0';
                    int box = (i/3)*3 + (j/3);
                    rows[i][num] = true;
                    cols[j][num] = true;
                    boxes[box][num] = true;
                }
            }
        }

        solve(board, rows, cols, boxes);
    }

    private boolean solve(char[][] board, boolean[][] rows, boolean[][] cols, boolean[][] boxes) {
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] == '.') {
                    int box = (i/3)*3 + (j/3);

                    for (int num = 1; num <= 9; num++) {
                        if (rows[i][num] || cols[j][num] || boxes[box][num]) continue;

                        // CHOOSE
                        board[i][j] = (char)('0' + num);
                        rows[i][num] = true;
                        cols[j][num] = true;
                        boxes[box][num] = true;

                        // EXPLORE
                        if (solve(board, rows, cols, boxes)) return true;  // stop if solved

                        // UNCHOOSE — only reached if path failed
                        board[i][j] = '.';
                        rows[i][num] = false;
                        cols[j][num] = false;
                        boxes[box][num] = false;
                    }

                    return false;  // no digit worked → tell parent to backtrack
                }
            }
        }
        return true;  // no empty cell found → board complete
    }
}
```

---

## The box index formula

```java
int box = (row / 3) * 3 + (col / 3);
```

The 9×9 board has nine 3×3 boxes, indexed 0–8:

```
box 0 | box 1 | box 2
box 3 | box 4 | box 5
box 6 | box 7 | box 8
```

Examples:
```
(0,0) → (0/3)*3 + (0/3) = 0   → box 0
(0,3) → (0/3)*3 + (3/3) = 1   → box 1
(3,0) → (3/3)*3 + (0/3) = 3   → box 3
(4,4) → (4/3)*3 + (4/3) = 4   → box 4
```

---

## Why array size is `[9][10]` not `[9][9]`

Digits are `1` to `9`, not `0` to `8`. Using digit directly as index:

```java
rows[r][9] = true;  // needs index 9 → size must be at least 10
```

Index `0` is simply never used — one wasted slot, no cost. Cleaner than storing `digit-1` everywhere.

---

## The three return situations — where to return what

```
Situation 1 — tried all 9 digits, none worked for this cell
→ return false   (tell parent: this path is dead, backtrack)

Situation 2 — scanned whole board, found zero empty cells
→ return true    (tell parent: board is complete, we're done)

Situation 3 — recursive call just returned true
→ return true immediately, skip unchoose
   (answer already found, don't undo anything)
```

```java
if (solve(...)) return true;   // situation 3 — propagate success up

return false;                  // situation 1 — no digit worked

// after the double loop:
return true;                   // situation 2 — no empty cell
```

---

## Why `void` breaks it — the critical lesson

With `void`, when the board completes the function just returns normally. The caller doesn't know if it succeeded — it continues the loop, hits unchoose, and **wipes the correct answer**.

```
void version:
  place 5 at (0,0)
    place 3 at (0,1)
      board complete → returns (void)
    ← caller doesn't know, runs unchoose → wipes 3  ❌
  ← runs unchoose → wipes 5  ❌
```

```
boolean version:
  place 5 at (0,0)
    place 3 at (0,1)
      board complete → return true
    ← true received → return true immediately  ✅
  ← true received → return true immediately  ✅
```

**Rule:** find-any problems need `boolean`. The moment a solution is found, `true` bubbles all the way up — unchoose is never reached because we already have the answer.

---

## My mistakes

### Bug 1 — wrong char conversion
```java
board[i][j] = (char) num;        // ❌ stores ASCII code 5, not character '5'
board[i][j] = (char)('0' + num); // ✅ '0' + 5 = '5'
```

### Bug 2 — void instead of boolean
```java
private void function(...)              // ❌ can't signal success
private boolean function(...)          // ✅ propagates true upward
```

### Bug 3 — pre-filling inside recursion
```java
// ❌ re-marks existing digits on every recursive call
} else {
    rows[i][num] = true; ...
}
```
Pre-fill existing digits **once** in the outer function before recursion starts, not inside the recursive call.

---

## Sudoku vs N-Queens

| | N-Queens | Sudoku |
|---|---|---|
| Place one per row | ✅ | ✅ (one digit per empty cell) |
| Column constraint | ✅ | ✅ |
| Diagonal constraint | ✅ | ❌ |
| Box constraint | ❌ | ✅ |
| Returns | all solutions | one solution |
| Return type | void | boolean |
| Find all vs find any | find all | find any |
