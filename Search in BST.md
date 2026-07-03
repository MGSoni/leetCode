# Search in BST — Notes

> **Problem:** Given a BST and a target value, return the subtree rooted at the node with that value. Return null if not found.
> LC 700 — leetcode.com/problems/search-in-a-binary-search-tree

---

## The pattern — binary search on a tree

At each node, compare target to current value:
- Equal → found, return this node
- Less → must be in left subtree, go left
- Greater → must be in right subtree, go right

You eliminate half the tree at every step — exactly like binary search on a sorted array, but on a tree structure.

---

## Solution

```java
class Solution {
    public TreeNode searchBST(TreeNode root, int val) {
        if (root == null) return null;           // not found

        if (root.val == val) return root;        // found — return subtree directly

        if (val < root.val)
            return searchBST(root.left, val);    // must be in left subtree
        else
            return searchBST(root.right, val);   // must be in right subtree
    }
}
```

---

## My mistake — not using BST property

```java
// ❌ searches both sides — ignores BST property
return search(root.left, val) || search(root.right, val);
```

This works but degrades to O(n) — same as searching a random unsorted tree. The BST property guarantees you never need to search both sides.

```java
// ✅ uses BST property — eliminates half the tree at every step
if (val < root.val) return searchBST(root.left, val);
else return searchBST(root.right, val);
```

**Complexity:**
- Without BST property: O(n) — visits every node
- With BST property: O(log n) for balanced BST, O(n) worst case for skewed tree

---

## My mistake — unnecessary node copying

```java
// ❌ manually copies the found node
result = new TreeNode();
result.val = root.val;
result.left = root.left;
result.right = root.right;
```

The problem asks to return the subtree rooted at the found node. You already have that — it's `root` itself. Just return it directly:

```java
if (root.val == val) return root;  // ✅ return the node, not a copy
```

---

## Is this "state passed down"?

No — and this distinction matters.

**State passed down** (valid BST): parent gives constraints to children that tighten as you go deeper. Children must stay within bounds set by ancestors.

**Search in BST**: `val` doesn't change at any level — you're searching for the same value throughout. The BST property tells you which direction to go, but nothing is being "passed down" in the sense of accumulated constraints.

The correct name for this pattern is **binary search on a tree** — eliminate half the search space at every node based on the BST ordering property.

---

## Trace

```
BST:
    4
   / \
  2   7
 / \
1   3

searchBST(4, 2)
  2 < 4 → go left
  searchBST(2, 2)
    2 == 2 → return node(2) with children [1,3] ✅
```

---

## BST property — why it enables O(log n)

```
Every node guarantees:
  all values in left subtree  < node.val
  all values in right subtree > node.val
```

At each node, one comparison tells you exactly which half of the remaining tree to search. You never need to look at the other half. For a balanced BST of n nodes, this gives O(log n) — same as binary search on a sorted array.

---

## One line to remember

> In a BST, go left if target is smaller, go right if larger — you never need to check both sides.
