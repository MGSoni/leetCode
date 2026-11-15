# Accounts Merge (Union-Find)

**Problem:** LeetCode 721. Given a list of accounts, each `[name, email1, email2, ...]`, merge accounts that share at least one email. Return merged accounts as `[name, sorted unique emails...]`, in any order. Two accounts can have the same name and still be different people.

**Link:** https://leetcode.com/problems/accounts-merge/

**Pattern:** Union-Find, grouping. Node = account index. A map from email to the first account that had it tells you when two accounts must be merged. After merging, group emails by leader.

---

## Core insight

- The thing that gets a leader is the **account** (its index), not the name and not the email. Same name does not mean same person, so the name can't be the key.
- Keep a `Map<String, Integer>` from **email to account index** (key = what you look up by, value = what you want back).
- For each email of each account:
  - email already in the map: union this account with the account the map gives
  - email not in the map: `map.put(email, thisAccount)`
- The map keeps the account where the email was **first seen**. It is never updated after a merge. `find()` later leads from that account to the real leader.
- After all merging, group: for each account, `leader = find(i)`, then add the account's emails to `Map<leader, TreeSet<email>>`.
- Build the answer: name from `accounts.get(leader).get(0)`, then the TreeSet's emails (already sorted, no duplicates).

## Plain-words plan

1. `parents[i] = i` for every account. Make `Map<String, Integer> emailAccountMap`.
2. For each account `i`, for each email (skip index 0, the name):
   - in map: `leaderA = find(mapAccount)`, `leaderB = find(i)`; if different, `parents[leaderB] = leaderA`
   - not in map: put `email -> i`
3. Second pass: for each account `i`, `leader = find(i)`, add all its emails to `mergedAccountMap[leader]` (a TreeSet).
4. For each entry: list = [name of leader's account] + sorted emails. Add to result.

## Hand trace

```
0: ["John", "a@x.com", "b@x.com"]
1: ["John", "b@x.com", "c@x.com"]
2: ["Mary", "d@x.com"]
3: ["John", "e@x.com"]
```

| Step | Action | parents | map |
|---|---|---|---|
| start | | [0,1,2,3] | {} |
| acc 0 | a, b new | [0,1,2,3] | a->0, b->0 |
| acc 1, b | in map (0): leaders 0 and 1 differ, parents[1] = 0 | [0,0,2,3] | |
| acc 1, c | new | [0,0,2,3] | c->1 |
| acc 2 | d new | [0,0,2,3] | d->2 |
| acc 3 | e new | [0,0,2,3] | e->3 |

Step 3 groups: `0 -> {a,b,c}`, `2 -> {d}`, `3 -> {e}`.

Output: `[John, a, b, c]`, `[Mary, d]`, `[John, e]`.

Interior case that caught a bug (three accounts, one person):

```
0: ["A", "a@x.com"]
1: ["A", "b@x.com"]
2: ["A", "a@x.com", "b@x.com"]
```

Account 2 links 0 and 1. With `parents[i] = leader` it wrote over account 2 twice and left 0 and 1 apart. With leaders: `a` gives `parents[2] = 0`, then `b` gives leaders 1 and 0, so `parents[0] = 1`. Final parents `[1,1,0]`. Account 2's real leader is `find(2)` = 1, not `parents[2]` = 0.

## Complexity

Let N = number of accounts, K = total number of emails across all accounts.

- **Time: O(K log K)** (plus O(N) for the array loops).
  - Step 2 visits each email once, with near-constant finds: O(K).
  - Step 3 inserts each email into a TreeSet once. One insert costs at most log K, because a set holds at most K items.
  - **Count the jobs:** K inserts, each at most log K, gives K log K. The outer loop over accounts and the inner loop over emails together visit every email once, so they add up to K. They do not multiply into K squared.
  - Step 4 copies the sorted sets out: O(K).
- **Space: O(N + K).** Parent array N, email map up to K, merged sets up to K, result K.
- If every one of the N accounts has about N emails, then K = N squared, which gives O(N^2 log N) time and O(N^2) space. Lead with the general K version in an interview.

**Revisit tomorrow:** re-derive "why K log K" from the count-the-jobs idea without looking.

## Final code

```java
class Solution {
    public List<List<String>> accountsMerge(List<List<String>> accounts) {
        int n = accounts.size();
        int[] parents = new int[n];
        for (int i = 0; i < n; i++) {
            parents[i] = i;                               // each account starts as its own leader
        }

        Map<String, Integer> emailAccountMap = new HashMap<>();   // email -> first account seen
        for (int i = 0; i < n; i++) {
            List<String> account = accounts.get(i);
            for (int j = 1; j < account.size(); j++) {            // skip the name at index 0
                String email = account.get(j);
                if (emailAccountMap.containsKey(email)) {
                    int leaderA = helper(emailAccountMap.get(email), parents);
                    int leaderB = helper(i, parents);
                    if (leaderA != leaderB) {
                        parents[leaderB] = leaderA;           // write to the LEADER
                    }
                } else {
                    emailAccountMap.put(email, i);
                }
            }
        }

        Map<Integer, TreeSet<String>> mergedAccountMap = new HashMap<>();   // leader -> sorted emails
        for (int i = 0; i < n; i++) {
            int leader = helper(i, parents);                  // true leader, not parents[i]
            TreeSet<String> sortedSet = mergedAccountMap.computeIfAbsent(leader, k -> new TreeSet<>());
            List<String> account = accounts.get(i);
            for (int j = 1; j < account.size(); j++) {
                sortedSet.add(account.get(j));
            }
        }

        List<List<String>> result = new ArrayList<>();
        for (Map.Entry<Integer, TreeSet<String>> entry : mergedAccountMap.entrySet()) {
            List<String> merged = new ArrayList<>();
            merged.add(accounts.get(entry.getKey()).get(0));  // name from the leader's account
            merged.addAll(entry.getValue());                  // emails, already sorted
            result.add(merged);
        }
        return result;
    }

    private int helper(int node, int[] parents) {
        int leader = node;
        while (parents[leader] != leader) {
            leader = parents[leader];
        }
        while (parents[node] != leader) {
            int next = parents[node];
            parents[node] = leader;
            node = next;
        }
        return leader;
    }
}
```

Your version had the same logic. The only differences here are `computeIfAbsent`, which replaces your if/else with two identical inner loops, and `addAll`, which replaces the loop that copies the set into the list. Both are optional tidy-ups.

## Mistakes and notes

| Item | What happened | Fix / rule |
|---|---|---|
| Template recall | Called find the "main function with the loop" | find is only the helper. The main loop calls it on both ends |
| Node choice | Considered name as the key | Same name does not mean same person. Node = account index, joined by shared email |
| Set vs map | Wanted a set of seen emails | A set says "seen", but you need **which account**. Map email -> account |
| Map direction | Said key = account, value = emails | Look up by email, get the account back: key = email, value = account |
| Merge write | `parents[i] = leader` | Find both leaders and write to a leader: `parents[leaderB] = leaderA` |
| Reading the leader | `leader = parents[i]` in step 3 | Not always the true leader. Call `helper(i, parents)` |
| Group key | Key = name for grouping | Key = leader, so three Johns do not get mixed |
| Compile slips | `leader` undefined, `map` vs `emailAccountMap`, `account.(j)`, `TreesSet`, step 3 inside step 2 loop, missing declaration | Re-read variable names before pasting. Check loop braces |
| Complexity | Reached n squared log n by multiplying loops | Count the jobs: K inserts, each at most log K. Loops add up, they do not multiply |

## Time taken

About two hours for a first-time Union-Find problem with a map and a group-by step. Redo it cold in a few days, with a 20-minute target.
