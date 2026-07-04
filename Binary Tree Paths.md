# Binary Tree Paths — Notes

> **Problem:** Given a binary tree, return all root-to-leaf paths as strings in "1->2->5" format.
> LC 257 — leetcode.com/problems/binary-tree-paths

---

## Why this was hard

This is the first problem where **backtracking and tree recursion are the same thing simultaneously**:

- Tree problems (height, sum) — no shared state, no unchoose needed
- Backtracking problems (subsets, permutations) — shared state, clear unchoose needed
- Binary Tree Paths — **shared mutable `temp` list** (backtracking) on a **tree structure** (tree recursion)

Both patterns active at once. The signal that backtracking is needed: a shared list passed into recursive calls.

---

## The three signals that mean "you need unchoose"

```
1. A list/array/string is passed into recursive calls
2. You add something to it before recursing
3. Multiple branches use the same object
```

All three present → write the unchoose before writing the recursive call.

---

## Two valid approaches

### Approach A — shared list, add on entry, remove on exit (preferred)

Every node adds itself on entry and removes itself on exit. `temp` is always clean when a new branch starts.

```java
class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> result = new ArrayList<>();
        function(root, new ArrayList<>(), result);
        return result;
    }

    private void function(TreeNode root, List<Integer> temp, List<String> result) {
        if (root == null) return;

        temp.add(root.val);                           // CHOOSE — every node, not just leaf

        if (root.left == null && root.right == null) {
            result.add(buildString(temp));            // leaf — record path
        } else {
            function(root.left, temp, result);        // EXPLORE left
            function(root.right, temp, result);       // EXPLORE right
        }

        temp.remove(temp.size()-1);                   // UNCHOOSE — every node, not just leaf
    }

    private String buildString(List<Integer> temp) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < temp.size(); i++) {
            sb.append(temp.get(i));
            if (i < temp.size()-1) sb.append("->");
        }
        return sb.toString();
    }
}
```

### Approach B — copied list, add before passing, no unchoose needed

Each branch gets its own copy — nothing shared, nothing to undo. Costs more memory.

```java
private void function(TreeNode root, List<Integer> temp, List<String> result) {
    if (root == null) return;

    List<Integer> newTemp = new ArrayList<>(temp);  // copy for this branch
    newTemp.add(root.val);

    if (root.left == null && root.right == null) {
        result.add(buildString(newTemp));
        return;
    }

    function(root.left, newTemp, result);
    function(root.right, newTemp, result);
    // no unchoose — newTemp is not shared
}
```

---

## The rule — pick one, don't mix

```
Shared list  → add on entry, remove on exit (every node, not just leaf)
Copied list  → add before passing down, no remove needed
```

**Mixing breaks it:** thinking "add only at leaf" (Approach B mindset) but passing a shared `temp` (Approach A structure) — values from one branch leak into the next branch.

---

## Trace for Approach A

```
Tree:
    1
   / \
  2   3
   \
    5

function(1, temp=[])
  add 1 → temp=[1]
  not leaf → explore
  function(2, temp=[1])
    add 2 → temp=[1,2]
    not leaf → explore
    function(null) → return
    function(5, temp=[1,2])
      add 5 → temp=[1,2,5]
      leaf → record "1->2->5" ✅
      remove 5 → temp=[1,2]
    remove 2 → temp=[1]        ← temp clean for right branch
  function(3, temp=[1])
    add 3 → temp=[1,3]
    leaf → record "1->3" ✅
    remove 3 → temp=[1]
  remove 1 → temp=[]

result = ["1->2->5", "1->3"] ✅
```

---

## My mistakes

### Mistake 1 — one temp shared across branches without unchoose

```java
temp.add(root.val);
function(root.left, temp);    // modifies temp
result.add(buildString(temp)); // temp has left branch values
// right branch never explored with clean temp
```

`temp` carried left branch values into right branch — paths got mixed.

### Mistake 2 — function explored both directions on same temp without unchoose

```java
function(root.left, temp);   // adds left values
function(root.right, temp);  // adds MORE to already-modified temp
// result: [1,2,4,5,3] instead of separate [1,2,4], [1,2,5], [1,3]
```

### Mistake 3 — `else if` missed right branch when left exists

```java
if (root.left != null) { ... }
else if (root.right != null) { ... }  // ❌ only runs if left is null
```

Both branches must be explored independently — `if/if` not `if/else if`.

---

## How this fits in the pattern map

| Problem | Structure | Shared state | Unchoose needed |
|---|---|---|---|
| height, sum | tree recursion | no | no |
| subsets, permutations | backtracking on array | yes | yes |
| Binary Tree Paths | backtracking ON a tree | yes | yes |
| grid paths | backtracking on grid | yes | yes |

Binary Tree Paths = same backtracking rules, just the structure you traverse is a tree instead of an array or grid.

---

## One line to remember

> Shared list on a tree → every node adds on entry, every node removes on exit. Not just the leaf — every single node.
