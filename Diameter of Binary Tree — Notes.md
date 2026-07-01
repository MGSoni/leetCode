# Diameter of Binary Tree — Notes

> **Problem:** Find the length of the longest path between any two nodes in a binary tree. The path may or may not pass through the root.
> LC 543 — leetcode.com/problems/diameter-of-binary-tree

---

## The pattern — external best-so-far

Some tree problems need to track **two different things simultaneously**:

1. **What to return to parent** — height of this subtree (parent needs this to compute its own diameter)
2. **The actual answer** — longest path seen anywhere in the tree (might not pass through the root at all)

These are different values serving different purposes. You can't return both from one function — so you track the answer in an **external variable** updated as a side effect, while the return value feeds the recursion normally.

```java
// return value  → height     → what parent needs
// side effect   → maxDiameter update → what the final answer needs
```

---

## Why height gives you diameter

`height(node)` = how far you can reach going **downward** from this node.

```
height(null) = 0
height(leaf) = 1
height(node) = Math.max(height(left), height(right)) + 1
```

The longest path **through** any node = how far you can reach going left + how far you can reach going right:

```
diameter at node = height(left child) + height(right child)
                 = "left reach"       + "right reach"
```

**Concrete example:**

```
        1
       / \
      2   3
     / \
    4   5

height(4) = 1, height(5) = 1, height(3) = 1
height(2) = Math.max(1,1)+1 = 2
height(1) = Math.max(2,1)+1 = 3

diameter at node 2 = height(4) + height(5) = 1+1 = 2  (path 4→2→5) ✅
diameter at node 1 = height(2) + height(3) = 2+1 = 3  (path 4→2→1→3) ✅
```

---

## Why you must check every node, not just root

The widest path might be deep in the tree, not at the root:

```
        1
       / \
      2   3
     / \
    4   5
   / \
  6   7

diameter at 4 = height(6)+height(7) = 1+1 = 2
diameter at 2 = height(4)+height(5) = 2+1 = 3
diameter at 1 = height(2)+height(3) = 3+1 = 4  ← maximum here happens to be at root

but in a different shaped tree, maximum could be deep inside
→ must update maxDiameter at EVERY node and keep running maximum
```

---

## Solution

```java
class Solution {
    int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        height(root);
        return maxDiameter;
    }

    private int height(TreeNode root) {
        if (root == null) return 0;

        int left = height(root.left);
        int right = height(root.right);

        maxDiameter = Math.max(maxDiameter, left + right);  // update global answer
        return Math.max(left, right) + 1;                   // return height to parent
    }
}
```

**The two lines that matter:**

```java
maxDiameter = Math.max(maxDiameter, left + right);  // diameter AT this node
return Math.max(left, right) + 1;                   // height OF this node
```

Same `left` and `right` values, two completely different purposes.

---

## How this differs from height and sum

| Problem | Return value | Extra tracking |
|---|---|---|
| height(node) | height | nothing |
| sum(node) | sum | nothing |
| isBalanced(node) | height or -1 sentinel | nothing — sentinel carries both signals |
| diameter(node) | height | external `maxDiameter` variable |

Diameter is the first problem where the **answer** and the **return value** are different things. Return value feeds the recursion. Answer lives outside it.

---

## The external variable pattern — when to use it

Use an external variable when:
- The answer is the **best value seen across all nodes** (maximum, minimum, count)
- But the **return value** needs to be something different to keep the recursion working

The external variable gets updated at every node as a side effect. The return value stays clean for the parent's computation.

---

## One line to remember

> Height measures reach downward. Diameter at a node is left reach + right reach. Track the maximum across all nodes externally because the widest path might be anywhere in the tree.
