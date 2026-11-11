# Merge k Sorted Lists

**Pattern:** K-way merge with a min-heap of list heads (linked-list version of [[kth-smallest-sorted-matrix-heap]])

## Problem
You are given an array of `k` linked lists, each sorted ascending. Merge them into one sorted linked
list and return it.

Example: `lists = [[1,4,5],[1,3,4],[2,6]]` -> `[1,1,2,3,4,4,5,6]`
Example: `lists = []` -> `[]`
Example: `lists = [[]]` -> `[]`

LeetCode 23: https://leetcode.com/problems/merge-k-sorted-lists/
Constraint: `k <= 10,000`, total number of nodes `N <= 10,000`

`k` = number of lists, `N` = total number of nodes across all lists.

## Approaches
| Approach | Time | Extra space | Status |
|---|---|---|---|
| Collect all values, sort, rebuild a list | O(N log N) | O(N) | Described, not coded |
| **Min-heap of list heads** | O(N log k) | O(k) | Solved (code below) |
| Merge lists in pairs, recursive | O(N log k) | O(log k) stack | Analysis done, not coded yet |
| Merge lists in pairs, in rounds (iterative) | O(N log k) | O(1) | Analysis done, not coded yet |
| Chain: merge 1 with 2, then with 3, and so on | O(N * k) | O(1) | Too slow, see below |

## Core insight (heap version)
Each linked list is a sorted "row", as in the sorted-matrix problem. Only the front node of each list
can be the next smallest, so the heap holds the current front node of each list.
- **The heap holds the nodes themselves**, compared by `node.val`. A node already knows its next node
  through `node.next`, so the entry does not need a row and a column.
- **After popping a node, push its `next` if it exists.** Push nothing if `next` is `null`.
- **Dummy head:** start the result with a dummy node and a `current` pointer. The first attach then looks
  the same as every other attach. After each attach, move `current` forward.
- **Always return `dummy.next`**, never the dummy itself. For `[]` and `[[]]` the heap stays empty,
  nothing is attached, and `dummy.next` is `null`, which is the empty list. No special case needed.
- **`PriorityQueue` throws on `null`.** Check each head for `null` before offering it.
- The nodes are relinked, not copied, so no new list is allocated.

## Algorithm
```
heap = min-heap ordered by node.val
for each list head in lists:
    if head != null: heap.offer(head)
dummy = new ListNode(); current = dummy
while heap is not empty:
    node = heap.poll()
    current.next = node
    current = current.next
    if node.next != null: heap.offer(node.next)
return dummy.next
```

## Hand trace, `[[1,4,5],[1,3,4],[2,6]]`
Heap starts with the heads `1`, `1`, `2`. Pop a `1` and push its `4`. Pop the other `1` and push its `3`.
Pop `2` and push its `6`. Pop `3` and push its `4`. The output begins `1, 1, 2, 3`.

## Complexity of the heap version
- **Time: O(N log k).** Every node is pushed once and popped once, so there are N pushes and N pops. Each
  costs O(log k), because the heap never holds more than `k` nodes. Only non-empty heads are offered.
- **Space: O(k).** The heap holds at most one node from each list. The cost of one heap operation is the log
  of the heap's size, and the space is the size itself, so it is O(k), not O(log k).

## Merging in pairs (analysis done, code still to do)
**Building block, merge two sorted lists:** compare the two current nodes, attach the smaller one to
`current`, and when one list runs out, attach the whole remainder of the other in one step (it is already
linked, so there is no need to walk it). For lists of `a` and `b` nodes it is O(a + b) time and O(1) extra
space, because it only relinks nodes. It has the same shape as the heap solution: dummy head, `current`
pointer.

**Pairs method:** merge lists 1 and 2, lists 3 and 4, and so on. Then merge those results in pairs again,
until one list is left.
- **Rounds:** the number of lists halves each round, so there are `log2(k)` rounds. For `k = 8`: 8 to 4, 4
  to 2, 2 to 1, which is 3 rounds. (There are 7 merges in total, but merges are not rounds.)
- **Cost per round:** in every round each node sits in exactly one merge, so one round walks `N` nodes in
  total. Round 1 has 4 merges of `N/4` each, round 2 has 2 merges of `N/2` each, round 3 has 1 merge of `N`.
- **Total: O(N log k).**
- **Space:** O(log k) for the recursive version (stack), O(1) for the iterative in-rounds version.

**Why not a chain:** in the chain, merge `i` walks about `i * (N/k)` nodes, because everything merged so far
is in it. The nodes of list 1 are walked about `k - 1` times. Adding it up gives about `N * k / 2`, which is
**O(N * k)**. With `k = 10,000` that is the difference between a few hundred thousand steps and tens of
millions.

**Still to do for the pairs version:**
- Plan the empty cases: `lists = []` should return `null`, and a `null` list is just an empty list.
- Decide what happens to the leftover list when `k` is odd.
- Trace on `[[1,4,5],[1,3,4],[2,6]]` (k = 3).
- Choose recursive (split the range in two, merge each half, then merge the results) or in-rounds.
- Code it using a `mergeTwo` helper.

## Solution (heap version)

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> minHeap = new PriorityQueue<>((a, b) -> Integer.compare(a.val, b.val));

        for (int i = 0; i < lists.length; i++) {
            if (lists[i] != null)
                minHeap.offer(lists[i]);
        }

        ListNode head = new ListNode();
        ListNode current = head;

        while (!minHeap.isEmpty()) {
            ListNode min = minHeap.poll();
            current.next = min;
            current = current.next;
            if (min.next != null) {
                minHeap.offer(min.next);
            }
        }

        return head.next;
    }
}
```

## Mistakes and confusions while working through it

| What went wrong | Fix / insight |
|---|---|
| Said to return "head itself, otherwise head next" | The dummy node is never part of the answer. Always return `dummy.next`, and the empty cases come out right with no special branch |
| Gave the space as O(log k) | That is the cost of one heap operation. The space is the heap's size, which is O(k) |
| Counted the rounds from 8 lists to 1 as 6 (`4 + 2`) | That counts merges. The number of rounds is `log2(k) = 3`, because the lists halve each round |
| Gave the per-round cost as `N/4`, then `N/2` | Those are the sizes of single merges. One whole round walks `N` nodes in total |
| Said list 1's nodes are walked `log k` times in a chain | They are walked about `k - 1` times, so the chain is O(N * k). The `log k` count belongs to the pairs method |
