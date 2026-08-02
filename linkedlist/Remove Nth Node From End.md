# Remove Nth Node From End — Notes

> **Problem:** Given the head of a linked list, remove the nth node from the end and return the head.
> LC 19 — leetcode.com/problems/remove-nth-node-from-end-of-list

---

## The core mental model — what every linked list recursive function does

> "Given this node as head, return the correctly modified list starting from this node."

When you recurse on `head.next`, you're saying:
> "Fix everything from `head.next` onwards. Give me back the head of that fixed sublist."

Then you have two things:
- `head` — current node, sitting in your hand
- result of recursion — correctly fixed sublist after head

Your job: **connect `head` to the fixed sublist and return `head`.**

---

## The universal linked list recursion template

```java
ListNode solve(ListNode head) {
    // 1. base case
    if (head == null) return null;
    // or: if (head.next == null) return head  ← when single node needs special handling

    // 2. recurse — fix everything after head
    head.next = solve(head.next);

    // 3. decide what to return
    // head should be KEPT    → return head
    // head should be REMOVED → return head.next
    // head changes role      → modify head, return something else
}
```

---

## The three decisions at the end of every function

```
Should head be kept?    → return head
Should head be removed? → return head.next
Does head change role?  → modify head, return something else (reverse only)
```

---

## Base case — null vs head.next == null

```
if (head == null) return null
→ use when single node has no special treatment
→ removeNthFromEnd, mergeTwoLists

if (head == null || head.next == null) return head
→ use when single node needs to be returned as-is
→ reverseList — single node is already reversed, becomes new head
```

**Why `head == null` returns `head` in reverse:**
`head == null` → `return head` → literally returns `null`. The two are identical when head is null. It's just a safety guard — the real base case in reverse is `head.next == null`.

---

## Why you wire result back to `head`

Before recursion:
```
head → [2] → [3] → [4] → [5] → null
```

After `removeNthFromEnd(head.next, n)` — fixes everything from node 2 onwards (say n=2, node 4 removed):
```
head still → [2]        [3] → [5] → null
                          ↑
                     fixed sublist starts here
```

`head.next` still points to old node 2. You must update it:

```java
head.next = removeNthFromEnd(head.next, n);  // wire fixed sublist to head
```

Now:
```
head → [3] → [5] → null  ✅
```

Then return `head` — it's still correct, just its `next` was updated.

---

## Solution

```java
class Solution {
    int count = 0;

    public ListNode removeNthFromEnd(ListNode head, int n) {
        if (head == null) return null;               // base case

        head.next = removeNthFromEnd(head.next, n);  // fix everything after head
        count++;                                      // count on the way back up

        if (count == n) return head.next;            // REMOVE this node — skip it
        return head;                                  // KEEP this node
    }
}
```

---

## Why count on the way BACK UP

You don't know which node is nth from end until you've reached the end. Counting on the way back gives you the position from the end naturally:

```
reach null  → count=0
at node 5   → count=1  (1st from end)
at node 4   → count=2  (2nd from end)  ← remove if n=2
at node 3   → count=3
...
```

---

## Slot trace — `1→2→3→4→5`, n=2

```
SLOTS: count | node | action

remove(1) → remove(2) → remove(3) → remove(4) → remove(5) → remove(null)
                                                               return null

remove(5): head.next=null,  count=1, 1!=2 → return 5
remove(4): head.next=5,     count=2, 2==2 → return head.next=5  ← node 4 skipped ✅
remove(3): head.next=5,     count=3, 3!=2 → return 3
remove(2): head.next=3→5,   count=4, 4!=2 → return 2
remove(1): head.next=2→3→5, count=5, 5!=2 → return 1

result: 1→2→3→5→null ✅
```

---

## My mistakes

### Bug 1 — wrong base case

```java
if (head.next == null) return head;  // ❌ single node has no special treatment here
if (head == null) return null;       // ✅ base case is null, not single node
```

### Bug 2 — lost head using temp

```java
ListNode temp = removeNthFromEnd(head.next, n);
// ❌ temp is head.next's result — head itself is lost
// never wired head.next, never returned head
```

```java
head.next = removeNthFromEnd(head.next, n);  // ✅ wire result to head.next
return head;                                  // ✅ return head
```

### Bug 3 — removing wrong node

```java
if (count == n) temp = temp.next;  // ❌ modifying temp, not removing head
if (count == n) return head.next;  // ✅ skip head by returning what it points to
```

---

## Template applied to all three linked list problems

```java
// Reverse — head changes role, returns newHead
ListNode reverse(ListNode head) {
    if (head == null || head.next == null) return head;  // single node = new head
    ListNode newHead = reverse(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;  // ← exception: doesn't return head
}

// Merge — picks smaller head each time
ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val <= l2.val) { l1.next = merge(l1.next, l2); return l1; }
    else                  { l2.next = merge(l1, l2.next); return l2; }
}

// Remove nth — keeps or skips head based on count
ListNode remove(ListNode head, int n) {
    if (head == null) return null;
    head.next = remove(head.next, n);
    count++;
    if (count == n) return head.next;  // REMOVE
    return head;                        // KEEP
}
```

---

## Four questions to answer before writing any linked list recursive function

```
1. What does this function return?
   → "head of correctly modified list starting from this node"

2. What is the base case?
   → null (nothing here) or single node (already done)?

3. What do I do with the recursion result?
   → wire it to head.next (most problems)
   → use as newHead (reverse only)

4. What do I return?
   → head (keep), head.next (remove), or newHead (reverse)
```

---

## One line to remember

> Recurse first, count on the way back. When count equals n, skip that node by returning head.next instead of head.
