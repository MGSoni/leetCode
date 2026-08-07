# Merge Two Sorted Lists — Notes

> **Problem:** Merge two sorted linked lists and return the merged list sorted.
> LC 21 — leetcode.com/problems/merge-two-sorted-lists

---

## The pattern — linear recursion on two lists

At every step answer one question: **"which node comes first?"**
Pick the smaller head, attach the merged remainder to it, return it.
No new nodes created — just rewire existing `next` pointers.

---

## Solution

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if (list1 == null) return list2;   // base case — one list exhausted
        if (list2 == null) return list1;   // base case — other list exhausted

        if (list1.val <= list2.val) {
            list1.next = mergeTwoLists(list1.next, list2);  // pick list1, merge rest
            return list1;
        } else {
            list2.next = mergeTwoLists(list1, list2.next);  // pick list2, merge rest
            return list2;
        }
    }
}
```

---

## 3-question recipe applied

**Base case:** one list is null → return the other. Nothing left to merge.

**Shrink:** pick the smaller head node, recurse on the remaining nodes.

**Combine:** attach merged remainder to the picked node's `next`, return the picked node.

---

## My mistake — unreachable code after early returns

```java
if (list1 == null && list2 == null) return new ListNode(); // ❌ wrong return value
else if (list1 == null) return list2;
else return list1;                                          // ❌ returns list1 always!

if (list1.val <= list2.val) { ... }   // ❌ NEVER REACHED — all paths above return
```

Two bugs in one:

**Bug 1 — `else return list1`** catches every case where `list1 != null` — including when `list2 != null` too. The merge logic below never runs. Every call returns `list1` immediately.

**Bug 2 — `return new ListNode()`** when both null returns an empty dummy node instead of `null`.

**The fix:** base cases should only handle null inputs, nothing else:

```java
if (list1 == null) return list2;   // ✅ only fires when list1 is null
if (list2 == null) return list1;   // ✅ only fires when list2 is null
// both non-null → falls through to merge logic below
```

**Rule:** in recursive functions, base cases must be narrow — only catch the exact condition they're meant for. An `else` that catches "everything remaining" kills the recursive case.

---

## Why only two base cases, not three

```java
if (list1 == null && list2 == null) return null;  // ❌ unnecessary
if (list1 == null) return list2;                  // ✅ handles both-null case too
if (list2 == null) return list1;                  // ✅
```

When both are null, `list1 == null` fires first and returns `list2` which is also null — correct result. No need for a separate both-null check.

---

## Slot trace — `1→2→4` and `1→3→4`

```
SLOTS: which node is picked and returned at each call

mergeTwoLists(1→2→4, 1→3→4)
  1<=1 → PICK list1's 1
  list1.next = mergeTwoLists(2→4, 1→3→4)
    2>1 → PICK list2's 1
    list2.next = mergeTwoLists(2→4, 3→4)
      2<=3 → PICK list1's 2
      list1.next = mergeTwoLists(4→null, 3→4)
        4>3 → PICK list2's 3
        list2.next = mergeTwoLists(4→null, 4→null)
          4<=4 → PICK list1's 4
          list1.next = mergeTwoLists(null, 4→null)
            list1==null → return list2's 4
          list1's 4 → next = list2's 4
          return list1's 4
        list2's 3 → next = 4→4→null
        return list2's 3
      list1's 2 → next = 3→4→4→null
      return list1's 2
    list2's 1 → next = 2→3→4→4→null
    return list2's 1
  list1's 1 → next = 1→2→3→4→4→null
  return list1's 1

RESULT: 1→1→2→3→4→4→null ✅
```

---

## Key insight — no new nodes, just rewiring

```java
list1.next = mergeTwoLists(list1.next, list2);
return list1;
```

You're not creating anything. You pick an existing node, point its `next` to the merged result of the remaining nodes, and return it. The entire merged list is built by rewiring existing `next` pointers on the way back up.

---

## How this maps to tree recursion patterns you know

| Tree | Linked list merge |
|---|---|
| `if (node == null) return 0` | `if (list == null) return other` |
| two recursive calls (left, right) | two recursive branches (pick list1 or list2) |
| combine on return | `node.next = result, return node` |
| base case = null node | base case = null list |

---

## Iterative vs recursive

**Recursive (this solution):**
```java
if (list1.val <= list2.val) {
    list1.next = mergeTwoLists(list1.next, list2);
    return list1;
}
```
Dives to the decision at each step, builds the list on the way back up. O(n+m) stack space.

**Iterative (dummy node approach):**
```java
ListNode dummy = new ListNode(0);
ListNode curr = dummy;
while (list1 != null && list2 != null) {
    if (list1.val <= list2.val) { curr.next = list1; list1 = list1.next; }
    else { curr.next = list2; list2 = list2.next; }
    curr = curr.next;
}
curr.next = (list1 != null) ? list1 : list2;
return dummy.next;
```
Uses a dummy head to avoid edge cases. O(1) space.

| | Recursive | Iterative |
|---|---|---|
| Space | O(n+m) stack | O(1) |
| Readability | cleaner, matches mental model | more explicit |
| Interview preferred | both acceptable | preferred for large lists |

---

## One line to remember

> Pick the smaller head, attach the merged remainder to its next, return it — no new nodes, just rewired pointers.
