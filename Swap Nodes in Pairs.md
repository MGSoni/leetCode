# Swap Nodes in Pairs — Notes

> **Problem:** Given a linked list, swap every two adjacent nodes and return the head.
> LC 24 — leetcode.com/problems/swap-nodes-in-pairs

---

## The pattern — swap pair, recurse on rest, wire together

At every step, swap the current pair and attach the swapped remainder.

```
Before: 1 → 2 → [rest]
After:  2 → 1 → [swapped rest]
```

Three things happen per pair:
1. Save second node (`temp = head.next`)
2. Point second → first (`temp.next = head`)
3. Point first → swapped remainder (`head.next = swapped`)
4. Return second as new head (`return temp`)

---

## Four questions applied

```
1. return: head of list with every pair swapped
2. base case: null (nothing to swap) or single node (can't swap) → return head
3. recursion result: wired to head.next (first node of pair points to swapped rest)
4. return: second node (temp) — it becomes the new head of this pair
```

---

## Solution

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        if (head == null || head.next == null) return head;  // base case

        ListNode swapped = swapPairs(head.next.next);  // recurse on rest (skip pair)
        ListNode temp = head.next;                     // save second node

        temp.next = head;      // second → first
        head.next = swapped;   // first → swapped remainder

        return temp;           // second is new head of this pair
    }
}
```

---

## Why skip TWO nodes in the recursive call

```java
swapPairs(head.next.next)  // skip current pair entirely
```

`head` = first node of current pair
`head.next` = second node of current pair
`head.next.next` = start of the next pair — that's where the next swap begins

If you only skip one (`head.next`), you'd be recursing into the second node of the current pair — the one you're about to move. Wrong.

---

## Slot trace — `1→2→3→4→null`

```
SLOTS: node connections at each step

swapPairs(1→2→3→4)
  swapPairs(3→4)
    swapPairs(null) → return null

    swapped = null
    temp = node4
    node4.next = node3    →  4→3
    node3.next = null     →  3→null
    return node4          →  4→3→null ✅

  swapped = 4→3→null
  temp = node2
  node2.next = node1      →  2→1
  node1.next = 4→3→null   →  1→4→3→null
  return node2            →  2→1→4→3→null ✅

result: 2→1→4→3→null ✅
```

---

## How this fits the universal template

```java
// universal template
ListNode solve(ListNode head) {
    if (base case) return ...;
    head.next = solve(head.next);   // fix remainder, wire to head
    return head;
}

// swap pairs — slight variation
ListNode swapPairs(ListNode head) {
    if (base case) return head;
    ListNode swapped = swapPairs(head.next.next);  // fix remainder (skip pair)
    ListNode temp = head.next;
    temp.next = head;          // swap the pair
    head.next = swapped;       // wire remainder to first node
    return temp;               // second node is new head
}
```

The difference from the standard template: you skip TWO nodes in the recursive call (not one), and the new head is `temp` (second node), not `head`.

---

## All five linked list problems — final side by side

```java
// REVERSE — new head floats up from base case
ListNode reverse(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverse(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}

// MERGE — pick smaller head each time
ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val <= l2.val) { l1.next = merge(l1.next, l2); return l1; }
    else                  { l2.next = merge(l1, l2.next); return l2; }
}

// PALINDROME — front forward, back unwinds backward
boolean check(ListNode back) {
    if (back == null) return true;
    boolean result = check(back.next);
    result = result && (front.val == back.val);
    front = front.next;
    return result;
}

// REMOVE NTH — count on way back, keep or skip
ListNode remove(ListNode head, int n) {
    if (head == null) return null;
    head.next = remove(head.next, n);
    count++;
    if (count == n) return head.next;
    return head;
}

// SWAP PAIRS — skip pair in recursion, swap, wire, return second
ListNode swapPairs(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode swapped = swapPairs(head.next.next);
    ListNode temp = head.next;
    temp.next = head;
    head.next = swapped;
    return temp;
}
```

---

## One line to remember

> Save second node, point second → first, point first → swapped rest, return second as new head.
