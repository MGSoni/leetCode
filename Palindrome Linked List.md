# Palindrome Linked List — Notes

> **Problem:** Given the head of a singly linked list, return true if it is a palindrome.
> LC 234 — leetcode.com/problems/palindrome-linked-list

---

## Why this is harder than array palindrome

With an array: compare `arr[0]` with `arr[n-1]`, move inward. Easy — random access.
With a linked list: can only go **forward**. Can't go backwards directly.

Two solutions — both worth knowing.

---

## Approach B — Recursive (recursion unwind trick)

### The insight

When recursion unwinds, it visits nodes in **reverse order**. So:
- `back` pointer moves backward (via recursion unwinding)
- `front` pointer moves forward (external variable, advances manually)
- Compare them at every step on the way back up

### Solution

```java
class Solution {
    ListNode front;

    public boolean isPalindrome(ListNode head) {
        front = head;
        return check(head);
    }

    private boolean check(ListNode back) {
        if (back == null) return true;              // reached end — start comparing

        boolean result = check(back.next);          // dive to the end first

        result = result && (front.val == back.val); // compare on the way back up
        front = front.next;                         // advance front forward
        return result;
    }
}
```

### Slot trace — `1→2→2→1`

```
SLOTS: front position | back position (unwinding)

check(back=1)              front=1
  check(back=2)            front=1
    check(back=2)          front=1
      check(back=1)        front=1
        check(null) → return true
      back=1, front=1 → 1==1 ✅, front→2,  return true
    back=2, front=2 → 2==2 ✅, front→2,  return true
  back=2, front=2 → 2==2 ✅, front→1,  return true
back=1, front=1 → 1==1 ✅, front→null, return true ✅
```

### Why `&&` with result matters

```java
result = result && (front.val == back.val);
```

Once any comparison fails (`result=false`), every subsequent call returns `false` immediately — short-circuit evaluation stops useless work. Same idea as `||` in word search.

---

## Approach C — Find middle + reverse second half (O(1) space)

### Three steps

```
Step 1 — find middle using slow/fast pointers
Step 2 — reverse second half (LC 206 — you already know this)
Step 3 — compare first half with reversed second half
```

### Step 1 — slow/fast pointer

```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;        // moves 1 step
    fast = fast.next.next;   // moves 2 steps
}
// when fast reaches end, slow is at middle
```

Fast moves twice as fast as slow. When fast reaches end, slow is at middle.

### Solution

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        if (head.next == null) return true;

        // step 1 — find middle
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }

        // step 2 — reverse second half
        ListNode reversed = reverse(slow);

        // step 3 — compare
        ListNode front = head, back = reversed;
        while (back != null) {
            if (front.val != back.val) return false;
            front = front.next;
            back = back.next;
        }

        return true;
    }

    private ListNode reverse(ListNode head) {
        if (head == null || head.next == null) return head;
        ListNode reversed = reverse(head.next);
        head.next.next = head;
        head.next = null;
        return reversed;
    }
}
```

### Slot trace — `1→2→2→1`

```
Step 1 — find middle:
  slow=1, fast=1
  iter 1: slow→2, fast→2(via next.next)
  iter 2: slow→2(second), fast→null
  middle = slow = second node 2

Step 2 — reverse(2→1):
  returns 1→2→null

Step 3 — compare:
  front=1, back=1 → 1==1 ✅, advance both
  front=2, back=2 → 2==2 ✅, advance both
  back=null → return true ✅
```

---

## My mistakes in Approach C

### Bug 1 — fast pointer crashes without null check

```java
while (fast != null) {
    fast = fast.next.next;  // ❌ crashes if fast.next is null
```

```java
while (fast != null && fast.next != null) {  // ✅
    fast = fast.next.next;
```

### Bug 2 — reverse returns null for single node

```java
if (head == null || head.next == null) return null;  // ❌
if (head == null || head.next == null) return head;  // ✅ single node is already reversed
```

### Bug 3 — wrong pointer in comparison

```java
if (head.val != reversed.val)  // ❌ head never advances — always compares first node
if (front.val != back.val)     // ✅ use dedicated front pointer
```

### Bug 4 — step 3 loop advances wrong

```java
fast = fast.next.next;  // ❌ skips every other node — confused step 1 with step 3
fast = fast.next;       // ✅ compare one node at a time
```

---

## Approach B vs Approach C

| | Approach B (recursive) | Approach C (reverse half) |
|---|---|---|
| Space | O(n) stack | O(1) |
| Modifies original list | no | yes — reverses second half |
| Elegance | recursion unwind trick | three-step pipeline |
| Interview signal | deep recursion understanding | pointer manipulation |
| Preferred when | space not critical | space matters |

---

## One line to remember

> **Approach B:** recursion unwinds in reverse — `back` goes backward, `front` goes forward, compare at every step.
> **Approach C:** find middle → reverse second half → compare both halves node by node.
