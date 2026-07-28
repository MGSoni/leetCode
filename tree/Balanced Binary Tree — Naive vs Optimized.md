# Balanced Binary Tree — Naive vs Optimized

> **Problem:** Given a binary tree, determine if it is height-balanced.
> LC 110 — leetcode.com/problems/balanced-binary-tree

---

## Solution 1 — Naive O(n²)

Call `height()` separately at every node. Simple to understand, but visits nodes multiple times.

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) return true;

        int leftHeight = height(root.left);
        int rightHeight = height(root.right);

        if (Math.abs(leftHeight - rightHeight) > 1) return false;

        return isBalanced(root.left) && isBalanced(root.right);
    }

    private int height(TreeNode node) {
        if (node == null) return 0;
        return 1 + Math.max(height(node.left), height(node.right));
    }
}
```

**What it does at every node:**
1. Call `height(left)` — traverses entire left subtree
2. Call `height(right)` — traverses entire right subtree
3. Check difference
4. Recurse into `isBalanced(left)` and `isBalanced(right)` — which call `height` again on the same subtrees

**Concrete trace:**

```
        1
       / \
      2   3
     /
    4

isBalanced(1)
  height(2) → visits 2,4      ← traversal 1
  height(3) → visits 3        ← traversal 2
  diff = |2-1| = 1 ✅
  isBalanced(2)
    height(4) → visits 4      ← traversal 3 — node 4 visited AGAIN
    height(null) → 0
    diff = 1 ✅
  isBalanced(3) → true
```

Node `4` visited twice — wasted work. For a skewed tree of n nodes:
```
Root calls height     → visits n nodes
Child calls height    → visits n-1 nodes
Next child            → visits n-2 nodes
...
Total = n + (n-1) + ... + 1 = O(n²)
```

---

## Solution 2 — Optimized O(n) using sentinel

One pass. Height computation and balance check happen simultaneously. The moment imbalance is detected, `-1` bubbles up instantly — no further computation.

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        return function(root) != -1;
    }

    private int function(TreeNode root) {
        if (root == null) return 0;                           // base case

        int leftHeight = function(root.left);
        if (leftHeight == -1) return -1;                      // left failed — stop immediately

        int rightHeight = function(root.right);
        if (rightHeight == -1) return -1;                     // right failed — stop immediately

        if (Math.abs(leftHeight - rightHeight) > 1) return -1;   // this node unbalanced

        return Math.max(leftHeight, rightHeight) + 1;         // normal height
    }
}
```

**Sentinel:** `-1` is impossible for a real height (heights always ≥ 0), so it's repurposed as a failure signal meaning "unbalanced detected below, stop computing."

**Same trace, optimized:**

```
        1
       / \
      2   3
     /
    4

function(4): left=0, right=0, diff=0 → return 1
function(2): left=1, right=0, diff=1 → return 2
function(3): left=0, right=0, diff=0 → return 1
function(1): left=2, right=1, diff=1 → return 3

Every node visited exactly once → O(n)
```

---

## Side by side comparison

| | Naive | Optimized |
|---|---|---|
| Time complexity | O(n²) | O(n) |
| Space complexity | O(h) | O(h) |
| Passes LeetCode | ✅ yes | ✅ yes |
| Visits each node | multiple times | exactly once |
| Two functions | yes — `isBalanced` + `height` | no — one function does both |
| Key idea | call height at every node separately | sentinel `-1` merges two return values |

---

## When to use which

**Naive** — when you need to explain your thought process clearly in an interview, or when the tree is small and O(n²) is acceptable. Easier to read and reason about.

**Optimized** — when the interviewer asks "can you do better?" or when n is large. The sentinel trick is the answer to any "compute X and validate Y simultaneously" tree problem.
