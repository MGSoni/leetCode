# Validate Binary Search Tree — Notes

> **Problem:** Given a binary tree, determine if it is a valid BST — every left subtree node is strictly less than the current node, every right subtree node is strictly greater.
> LC 98 — leetcode.com/problems/validate-binary-search-tree

---

## The pattern — state passed DOWN into calls

All previous tree problems had information flowing **upward** — children computed something, returned it, parent combined it.

Valid BST is the first problem where information flows **downward**. The root tells its children what bounds they must stay within. Each child passes tighter bounds to its own children.

```
root(5) → left child must be in (-∞, 5)
        → right child must be in (5, +∞)

left(3) → its left child must be in (-∞, 3)
        → its right child must be in (3, 5)   ← both root AND parent constraints combined
```

The bounds tighten as you go deeper. This tightening is the entire algorithm.

---

## Solution

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValid(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    private boolean isValid(TreeNode root, long min, long max) {
        if (root == null) return true;                          // base case

        if (root.val <= min || root.val >= max) return false;  // out of bounds

        return isValid(root.left, min, root.val)               // left: must be < current
            && isValid(root.right, root.val, max);             // right: must be > current
    }
}
```

**The two recursive calls:**

```java
isValid(root.left,  min,      root.val)  // left child: upper bound tightens to current val
isValid(root.right, root.val, max)       // right child: lower bound tightens to current val
```

Every call carries the accumulated constraints from all ancestors above it.

---

## Why the naive approach fails

```java
// WRONG — only checks immediate children, not all descendants
boolean isValidBST(TreeNode node) {
    if (node == null) return true;
    if (node.left != null && node.left.val >= node.val) return false;
    if (node.right != null && node.right.val <= node.val) return false;
    return isValidBST(node.left) && isValidBST(node.right);
}
```

Fails on:
```
    5
   / \
  1   4
     / \
    3   6
```

Node `4` has locally valid children (`3 < 4 < 6`). But `4` is in the right subtree of `5` — it must be `> 5`. The naive check never catches violations involving ancestors, only immediate parents.

**Fix:** pass the accumulated min/max bounds from every ancestor down through the recursion.

---

## Why `Long` not `Integer`

Node values can be `Integer.MIN_VALUE` or `Integer.MAX_VALUE`. If you used `Integer.MIN_VALUE` as the initial lower bound:

```java
if (root.val <= min)  // root.val = Integer.MIN_VALUE, min = Integer.MIN_VALUE
                      // Integer.MIN_VALUE <= Integer.MIN_VALUE → true → returns false ❌
```

A valid node with value `Integer.MIN_VALUE` would be wrongly rejected. `Long.MIN_VALUE` is safely outside the integer range — no real node value can equal it.

---

## Trace on valid tree

```
    5
   / \
  3   7
 / \
1   4

isValid(5, -∞, +∞)
  5 in (-∞, +∞) ✅
  isValid(3, -∞, 5)
    3 in (-∞, 5) ✅
    isValid(1, -∞, 3)
      1 in (-∞, 3) ✅ → true
    isValid(4, 3, 5)
      4 in (3, 5) ✅ → true
  isValid(7, 5, +∞)
    7 in (5, +∞) ✅ → true

result: true ✅
```

## Trace on invalid tree

```
    5
   / \
  1   4    ← 4 < 5 but in right subtree

isValid(5, -∞, +∞)
  5 in (-∞, +∞) ✅
  isValid(1, -∞, 5) → true
  isValid(4, 5, +∞)
    4 in (5, +∞)? NO → return false ❌

result: false ✅
```

---

## How information flows — all four tree patterns compared

| Problem | Information flows | Key idea |
|---|---|---|
| height, sum | upward only | children return values, parent combines |
| isBalanced | upward with sentinel | `-1` smuggles failure signal in return value |
| diameter | upward + external | return height to parent, update maxDiameter as side effect |
| valid BST | downward | parent passes bounds to children, children pass tighter bounds further down |

Valid BST is the only one where the parent **gives** something to the child, instead of the child giving something to the parent.

---

## One line to remember

> Pass `(min, max)` bounds down — each node must be strictly inside them, left child tightens the max, right child tightens the min.
