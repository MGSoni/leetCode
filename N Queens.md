# N-Queens — Notes

> **Problem:** Place n queens on an n×n chessboard so no two queens attack each other. Return all valid arrangements.
> LC 51 — leetcode.com/problems/n-queens

---

## The pattern

N-Queens = backtracking where you place **one queen per row**, top to bottom.

At each row, try every column. Skip columns that are under attack. Place, recurse to next row, unplace.

---

## What "safe" means for cell (row, col)

A queen attacks in 3 directions — same column, `\` diagonal, `/` diagonal.

Track three boolean arrays:

| Array | Tracks | Index |
|---|---|---|
| `usedCols` | columns under attack | `col` |
| `diag1` | `\` diagonals under attack | `row + col` |
| `diag2` | `/` diagonals under attack | `row - col + n` (offset to avoid negatives) |

**Why `row+col` identifies `\` diagonals:** every cell on the same `\` diagonal shares the same `row+col` value.

**Why `row-col` identifies `/` diagonals:** every cell on the same `/` diagonal shares the same `row-col` value.

**Why offset by `+n` for diag2:** `row-col` ranges from `-(n-1)` to `+(n-1)` — negative indices crash. Adding `n` shifts range to `1` to `2n-1`, all positive. So array size = `2*n`.

---

## Solution

```java
class Solution {
    public List<List<String>> solveNQueens(int n) {
        List<List<String>> result = new ArrayList<>();
        boolean[] usedCols = new boolean[n];
        boolean[] diag1 = new boolean[2 * n];   // row+col
        boolean[] diag2 = new boolean[2 * n];   // row-col+n
        solve(0, n, usedCols, diag1, diag2, new ArrayList<>(), result);
        return result;
    }

    private void solve(int row, int n, boolean[] usedCols, boolean[] diag1, boolean[] diag2,
                       List<String> board, List<List<String>> result) {
        if (row == n) {
            result.add(new ArrayList<>(board));  // base case — all n queens placed
            return;
        }

        for (int col = 0; col < n; col++) {
            if (usedCols[col] || diag1[row + col] || diag2[row - col + n]) continue;

            // CHOOSE
            usedCols[col] = true;
            diag1[row + col] = true;
            diag2[row - col + n] = true;
            board.add(buildRow(col, n));

            // EXPLORE
            solve(row + 1, n, usedCols, diag1, diag2, board, result);

            // UNCHOOSE
            usedCols[col] = false;
            diag1[row + col] = false;
            diag2[row - col + n] = false;
            board.remove(board.size() - 1);
        }
    }

    private String buildRow(int queenCol, int n) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < n; i++) {
            sb.append(i == queenCol ? 'Q' : '.');
        }
        return sb.toString();
    }
}
```

---

## The symmetry rule — golden rule of backtracking

Every choose line has an exact mirror in unchoose, in reverse order:

```
CHOOSE                          UNCHOOSE
──────────────────────────────────────────
usedCols[col] = true    ↔   usedCols[col] = false
diag1[...] = true       ↔   diag1[...] = false
diag2[...] = true       ↔   diag2[...] = false
board.add(...)          ↔   board.remove(...)
```

If your choose and unchoose aren't perfectly symmetric, state leaks into other branches — wrong answers with no crash.

---

## My original approach vs correct approach

**My first instinct:** use a 2D safe matrix — mark cells as safe/unsafe on each placement.

**Why it doesn't work cleanly:** when backtracking, you'd need to restore exactly which cells were newly marked unsafe by this placement vs already unsafe before it. That requires storing the entire matrix state per call — expensive and complex.

**Better approach:** track only 3 attacked lines (column, two diagonals). Undo is trivial — just flip the boolean back. O(1) choose and unchoose.

---

## Trace for n=4, first valid placement

```
row=0: try col=0,1,2,3
  col=1: place Q → board=["...Q..."], usedCols[1]=T, diag1[1]=T, diag2[4]=T
    row=1: try col=0,1,2,3
      col=0: diag2[1-0+4=5]? no, diag1[1+0=1]? YES → skip
      col=1: usedCols[1]=T → skip
      col=2: safe → place Q
        row=2: ...continues until full board found
```

---

## Complexity

- Time: O(n!) — you're generating all valid arrangements, unavoidable
- Space: O(n) — three arrays of size n or 2n, plus board of n strings
