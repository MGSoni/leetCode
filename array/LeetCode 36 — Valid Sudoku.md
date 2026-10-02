# LeetCode 36 — Valid Sudoku

**Pattern:** Hashmap/Set-based duplicate detection across 3 overlapping groupings (row, column, 3x3 box)
**Difficulty:** Medium
**Link:** https://leetcode.com/problems/valid-sudoku/

---

## Problem
Given a 9x9 Sudoku board (partially filled, empty cells marked `.`), determine if it's valid: no row, column, or 3x3 sub-box contains a repeated digit 1-9. Only the filled cells need to be validated — the board doesn't need to be solvable.

---

## My Solution (final, after fixing the empty-cell bug)
```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        
        Map<Integer,List<Character>> rowMap = new HashMap<>();
        Map<Integer,List<Character>> colMap = new HashMap<>();
        Map<Integer,List<Character>> boxMap = new HashMap<>();
        List<Character> valueList = List.of('1','2','3','4','5','6','7','8','9');
        for(int i=0;i<9;i++){
            for(int j=0;j<9;j++){
                char value = board[i][j];
                if (value == '.') continue;
                if(!valueList.contains(value)){
                    return false;
                }
                
                int box = (i/3)*3 + (j/3);
                //row 
                if(rowMap.containsKey(i)){
                    List<Character> rowList = rowMap.get(i);
                    if(rowList.contains(value)){
                        return false;
                    }else{
                        rowList.add(value);
                    }
                }else{
                    List<Character> rowList = new ArrayList<>();
                    rowList.add(value);
                    rowMap.put(i,rowList);
                }
                //col check
                if(colMap.containsKey(j)){
                    List<Character> colList = colMap.get(j);
                    if(colList.contains(value)){
                        return false;
                    }else{
                        colList.add(value);
                    }
                }else{
                    List<Character> colList = new ArrayList<>();
                    colList.add(value);
                    colMap.put(j,colList);
                }
                //box check
                if(boxMap.containsKey(box)){
                    List<Character> boxList = boxMap.get(box);
                    if(boxList.contains(value)){
                        return false;
                    }else{
                        boxList.add(value);
                    }
                }else{
                    List<Character> boxList = new ArrayList<>();
                    boxList.add(value);
                    boxMap.put(box,boxList);
                }
            }
        }
        return true;
    }
}
```
**Verdict:** ✅ Correct after fix. O(1) per-cell work (9 possible values per group) → O(81) ≈ O(1) overall for a fixed 9x9 board. Space: O(1) bounded (at most 9 rows × 9 cols × 9 boxes, each holding ≤9 values).

**Box index formula:** `box = (i/3)*3 + (j/3)` — maps each cell to one of 9 box indices (0-8) by dividing the grid into 3x3 blocks using integer division.

---

## The Bug (first version)
Original code checked `if(!valueList.contains(value)) return false;` **before** skipping empty cells. Since `.` is not in `valueList`, this returned `false` on the very first empty cell encountered — which is almost every cell on a real Sudoku board (boards are mostly empty by definition). The fix: explicitly `continue` on `'.'` before running any validity checks.

**Lesson:** when a grid/array can contain a placeholder/sentinel value (like `.` for empty), it must be explicitly filtered out *before* any validation logic — don't let it fall through into checks meant only for real values.

---

## Possible Improvement (not a correctness bug)
Using `List<Character>` + `.contains()` is an O(n) linear scan per duplicate check. Switching to `Set<Character>` makes it O(1) via `.add()`'s return value (returns `false` if the element was already present, combining the check and insert into one call):

```java
Map<Integer, Set<Character>> rowMap = new HashMap<>();
...
rowMap.putIfAbsent(i, new HashSet<>());
if (!rowMap.get(i).add(value)) return false;
```

Negligible performance difference here (max list size is 9), but the right default habit for "have I seen this before" checks — same principle as using a hashmap over a list in Subarray Sum Equals K (560).

---

## Mistakes / Things to Remember
- **Explicitly skip sentinel/placeholder values (`.`) before running validity checks on real data** — don't let them fall through into logic meant only for actual values. Same category as the Edit Distance memo sentinel bug (0 used as both "uncomputed" and "valid answer") — a recurring personal pattern: conflating or mishandling placeholder/sentinel values.
- Prefer `Set.add()`'s boolean return over `List.contains()` + separate `.add()` for duplicate-detection — fewer lines, O(1) instead of O(n).

## Related Problems
- Sudoku Solver (37) — backtracking version of this same validation logic
- Valid Anagram (242) — similar duplicate/frequency tracking via hashmap
