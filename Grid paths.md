# Grid Path Finding — Notes

> **Problem:** Find all paths from top-left `(0,0)` to bottom-right `(m-1,n-1)` of a grid, moving only right or down.
> No LeetCode link — pure warm-up before Word Search

---

## The pattern

Grid recursion = backtracking where you **move through a 2D grid** one cell at a time.

At each cell, try all valid moves (right, down). Add current cell to path, recurse, remove on return.

---

## Solution

```java
void findPaths(int row, int col, int m, int n,
               List<String> path, List<List<String>> result) {

    if (row == m-1 && col == n-1) {            // destination — check FIRST
        path.add("(" + row + "," + col + ")");
        result.add(new ArrayList<>(path));
        path.remove(path.size()-1);
        return;
    }

    if (row >= m || col >= n) return;          // out of bounds — check SECOND

    path.add("(" + row + "," + col + ")");     // CHOOSE

    findPaths(row+1, col, m, n, path, result); // EXPLORE down
    findPaths(row, col+1, m, n, path, result); // EXPLORE right

    path.remove(path.size()-1);                // UNCHOOSE
}
```

---

## My mistakes and why they happen

### Bug 1 — bounds check before destination check

```java
if (row >= m || col >= n) return;       // ❌ fires first
if (row == m-1 && col == n-1) ...       // never reached
```

When you arrive at `(m-1, n-1)`, the bounds check fires first and returns early — destination never recorded.

**Fix — always check destination before bounds:**

```java
if (row == m-1 && col == n-1) { ... }  // ✅ destination first
if (row >= m || col >= n) return;       // ✅ bounds second
```

**Rule:** in grid problems, destination check always comes before bounds check.

### Bug 2 — missing unchoose

```java
path.add("("+row+","+col+")");
findPaths(row+1, col, ...);
findPaths(row, col+1, ...);
// ← missing path.remove() here
```

`path` is shared across all branches. Without removing the current cell after both recursive calls return, it bleeds into sibling branches — wrong paths recorded.

**Fix:** always remove after all recursive calls return:

```java
path.add("("+row+","+col+")");
findPaths(row+1, col, ...);
findPaths(row, col+1, ...);
path.remove(path.size()-1);  // UNCHOOSE — always
```

---

## Trace for 2x2 grid (m=2, n=2)

```
findPaths(0,0)
  path=[( 0,0)]
  down → findPaths(1,0)
    path=[(0,0),(1,0)]
    down → findPaths(2,0) → row>=m → return
    right → findPaths(1,1) → destination!
      record [(0,0),(1,0),(1,1)] ✅
    unchoose → path=[(0,0)]
  right → findPaths(0,1)
    path=[(0,0),(0,1)]
    down → findPaths(1,1) → destination!
      record [(0,0),(0,1),(1,1)] ✅
    right → findPaths(0,2) → col>=n → return
    unchoose → path=[(0,0)]
  unchoose → path=[]

result = [[(0,0),(1,0),(1,1)], [(0,0),(0,1),(1,1)]]
```

---

## Choose once vs choose per direction

Both of these work:

**Version A — choose once, explore all directions, unchoose once:**
```java
path.add("("+row+","+col+")");         // CHOOSE once
findPaths(row+1, col, m, n, path, result);  // explore down
findPaths(row, col+1, m, n, path, result);  // explore right
path.remove(path.size()-1);            // UNCHOOSE once
```

**Version B — choose/unchoose around each direction separately:**
```java
path.add("("+row+","+col+")");
findPaths(row+1, col, m, n, path, result);  // explore down
path.remove(path.size()-1);

path.add("("+row+","+col+")");
findPaths(row, col+1, m, n, path, result);  // explore right
path.remove(path.size()-1);
```

**Same output. Version A is cleaner.** Version B adds and removes the same cell once per direction — for 2 directions that's manageable, but for Word Search with 4 directions it becomes 4 adds and 4 removes of the same cell. Version A handles any number of directions with one add and one remove.

**The rule:** choose/unchoose wraps around **what you add to shared state** — not around each recursive call individually. You're adding the cell once, so choose/unchoose once after all directions are explored.

---

## Key rules for grid recursion

1. **Destination check before bounds check** — always
2. **Unchoose after all recursive calls** — path is shared state
3. **No visited matrix needed here** — moving only right/down means you can never revisit a cell naturally
4. **Each direction = one recursive call** — 2 directions here, Word Search has 4

---

## How this connects to Word Search

| | Grid paths | Word Search |
|---|---|---|
| Directions | right, down (2) | up, down, left, right (4) |
| Stop condition | reached destination | matched all characters |
| Extra check | none | character must match |
| Visited tracking | not needed | needed — can't reuse cells |
| Returns | void, collects all paths | boolean, stops at first match |

Word Search is this problem with 4 directions, a character match condition, and a visited matrix.
