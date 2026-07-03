# Path Sum — Notes

> **Problem:** Given a binary tree and a target sum, return true if there exists a root-to-leaf path where the node values sum to target.
> LC 112 — leetcode.com/problems/path-sum

---

## The pattern — two patterns combined

This problem combines two patterns you already know:

1. **State passed down** (like valid BST) — subtract current node's value from target as you go deeper
2. **Boolean propagated up** (like word search) — return `true` the moment any path succeeds

```java
// state flows DOWN — target shrinks at every level
hasPathSum(root.left, targetSum - root.val)

// boolean flows UP — true bubbles up through ||
return hasPathSum(left, ...) || hasPathSum(right, ...)
```

---

## Solution

```java
class Solution {
    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) return false;                         // base case 1 — overshot

        if (root.left == null && root.right == null)           // base case 2 — leaf reached
            return targetSum == root.val;

        return hasPathSum(root.left,  targetSum - root.val)    // explore left
            || hasPathSum(root.right, targetSum - root.val);   // explore right
    }
}
```

---

## Two base cases — why both are needed

```java
if (root == null) return false;
```
Handles the case where you go past a leaf (e.g. a leaf's null child). Returns false — no path here.

```java
if (root.left == null && root.right == null)
    return targetSum == root.val;
```
Handles reaching an actual leaf. Check if the remaining target equals this leaf's value — if yes, path found.

**Why not just use `root == null`?**

If you only had the null check, you'd subtract the leaf's value and recurse into both null children — both return false, even if the path was valid. The leaf check catches success at the right moment.

---

## Trace

```
Tree:
        5
       / \
      4   8
     /   / \
    11  13   4
   /  \       \
  7    2       1

targetSum = 22

hasPathSum(5, 22)
  hasPathSum(4, 17)
    hasPathSum(11, 13)
      hasPathSum(7, 2)
        leaf, 2 == 7? false
      hasPathSum(2, 2)
        leaf, 2 == 2? true ✅  ← path: 5→4→11→2
      returns true
    returns true
  returns true ✅
```

---

## What patterns this problem tested

| Pattern | Where used |
|---|---|
| Two base cases | null check + leaf check |
| State passed down | `targetSum - root.val` shrinks each level |
| Boolean propagated up | `\|\|` stops at first true |
| Binary tree recursion | two recursive calls, left and right |

---

## Key insight

You don't need to track a running sum going upward. Instead, **subtract as you go down** — by the time you reach a leaf, the remaining target should equal exactly the leaf's value. Simpler than accumulating a sum upward and comparing at the top.

---

## One line to remember

> Subtract current node's value from target going down. At a leaf, if remaining target equals leaf value — path found.
