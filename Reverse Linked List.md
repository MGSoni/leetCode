# Reverse Linked List — Notes

> **Problem:** Reverse a singly linked list recursively.
> LC 206 — leetcode.com/problems/reverse-linked-list

---

## Linked list vs tree — the connection

A linked list is just a tree where every node has exactly one child:

```java
// tree node — two children
class TreeNode {
    int val;
    TreeNode left, right;
}

// linked list node — one child
class ListNode {
    int val;
    ListNode next;   // ← just one child
}
```

Everything you know about tree recursion applies directly.
Base case = null node. Shrink = move to next node. Combine = fix connections on return.

---

## What `reverseList(node)` returns

It returns **the head of the fully reversed list** starting from `node`.

Trust this contract completely — same leap of faith as `sum(n-1)` returning the correct sum. You don't trace the whole recursion. You just ask: "if the smaller problem is already solved, what do I do with what I have?"

---

## The key insight

`reverseList(head.next)` returns the new head of the reversed list.
You don't touch it. You just fix `head`'s own connections and pass the new head upward unchanged.

The new head "floats" up through every recursive call untouched:

```
reverseList(3) → returns node 3   (base case)
reverseList(2) → fixes node 2, returns node 3
reverseList(1) → fixes node 1, returns node 3
```

---

## Solution

```java
public ListNode reverseList(ListNode head) {
    // base case — empty or single node, already reversed
    if (head == null || head.next == null) return head;

    // trust — reverse everything after head
    ListNode newHead = reverseList(head.next);

    // combine — fix head's connections (two lines)
    head.next.next = head;   // successor points back to head
    head.next = null;        // head points to nothing — it's now the tail

    // pass new head up unchanged
    return newHead;
}
```

---

## The two combine lines — why each one

**`head.next.next = head`**

After `reverseList(head.next)`, the sublist is reversed but `head.next` (your successor) is now the tail — its `next` is null. You need it to point back at `head`:

```
Before: 1 → 2 ← 3 ← 4
                ↑
           head.next = node 2 (tail of reversed sublist)

head.next.next = head  →  node 2 → node 1
```

**`head.next = null`**

Without this, node 1 still points forward to node 2 — creating a cycle:

```
1 ⇄ 2 ← 3 ← 4   ← cycle! infinite loop
```

Cutting it makes node 1 the proper tail:

```
4 → 3 → 2 → 1 → null   ✅
```

---

## Slot trace — `1 → 2 → 3 → null`

```
SLOTS: connections between nodes

reverseList(1)
  reverseList(2)
    reverseList(3)
      head.next==null  →  BASE CASE, return node 3

    newHead = node 3
    SLOTS before: 1→2→3→null, reversed part: 3→null, node 2 still →null
    head.next.next = head  →  node 3→node 2, node 2→node 3  (wait, node 2 is head here)

    head=2:
      head.next = node 3
      head.next.next = head  →  node 3 → node 2
      head.next = null       →  node 2 → null
    SLOTS: 3→2→null,  node 1 still →node 2
    return node 3

  newHead = node 3
  head=1:
    head.next = node 2
    head.next.next = head  →  node 2 → node 1
    head.next = null       →  node 1 → null
  SLOTS: 3→2→1→null  ✅
  return node 3

RESULT: head = node 3, list = 3→2→1→null  ✅
```

---

## Iterative vs recursive — same goal, different mental model

**Iterative (prev/curr pointers):**
```java
ListNode prev = null, curr = head;
while (curr != null) {
    ListNode next = curr.next;
    curr.next = prev;          // flip pointer
    prev = curr;               // move forward
    curr = next;
}
return prev;
```
Moves forward through the list, flipping as it goes.

**Recursive:**
```java
ListNode newHead = reverseList(head.next);  // dive to the end first
head.next.next = head;                      // fix on the way back
head.next = null;
return newHead;
```
Dives to the end first, fixes connections on the way back up.

| | Iterative | Recursive |
|---|---|---|
| Direction | forward → flip | dive to end → fix on return |
| Space | O(1) | O(n) stack space |
| Easier to reason about | pointer manipulation | trust + combine |

---

## One line to remember

> Reverse the rest, make your successor point back at you, cut your forward pointer, pass the new head up unchanged.
