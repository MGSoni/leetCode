# Word Break — Notes

> **Problem:** Given a string s and a dictionary of words, return true if s can be segmented into dictionary words.
> LC 139 — leetcode.com/problems/word-break

---

## The pattern — recursion with loop → DP (boolean)

At every position `start`, try every possible word ending at `i`. If the word exists in dictionary AND the rest of the string can also be broken → return true.

```
solve(start) = true if ANY i exists where:
  1. s.substring(start, i) is in dictionary
  2. solve(i) is also true
```

---

## My incorrect solutions and what was wrong

### Attempt 1 — if/else instead of loop

```java
private boolean solve(String s, List<String> wordDict, int partition) {
    if (partition >= s.length()) {
        if (wordDict.contains(s)) return true;
        return false;
    }

    if (wordDict.contains(s.substring(0, partition))) {
        return solve(s.substring(partition), wordDict, 1);  // ❌
    } else {
        return solve(s, wordDict, partition+1);             // ❌
    }
}
```

**What was wrong:**
- `if/else` structure — if first valid word leads to dead end, gives up instead of trying longer partitions
- Only tries ONE partition per call, not all of them
- `s.substring(0, partition)` — always checks from 0, ignores that start has moved

**The fix:** replace `if/else` with a `for` loop. Try ALL partitions, return true only if the recursive call also succeeds.

---

### Attempt 2 — loop but wrong substring and index

```java
for (int i = 1; i <= s.length(); i++) {
    if (wordDict.contains(s.substring(0, i))) {      // ❌ always from 0
        return solve(s.substring(i+1), wordDict);    // ❌ skips a character
    }
}
```

**What was wrong:**
- `s.substring(0, i)` — always checks from position 0, not from `start`
- `s.substring(i+1)` — skips character at position `i`
- Returns immediately on first valid word without trying others

---

### Attempt 3 — wrong memo type

```java
boolean[] memo;   // ❌ can't represent three states

private boolean solve(...) {
    if (memo[start]) return true;   // ❌ misses false cases — recomputes unnecessarily
    ...
    if (memo[i] == true) return true;  // ❌ double-checking unnecessarily
    ...
}
```

**What was wrong:**
- `boolean[]` has only `true` and `false` — no way to say "not computed yet"
- `if (memo[start]) return true` — only catches cached `true`, ignores cached `false`
- This means positions that definitely CAN'T break get recomputed every time

**The fix:** use `Boolean[]` — null means "not computed", true/false means "computed result."

---

## Correct solution — plain recursion

```java
private boolean solve(String s, List<String> wordDict, int start) {
    if (start == s.length()) return true;   // fully matched ✅

    for (int i = start+1; i <= s.length(); i++) {        // try every end position
        if (wordDict.contains(s.substring(start, i))) {  // valid word found?
            if (solve(s, wordDict, i)) return true;       // rest also breakable?
        }
    }

    return false;  // no valid partition found
}
```

---

## Correct solution — memoization

**Use `Boolean[]` not `boolean[]`:**
```
null  = not computed yet
true  = this position CAN be broken
false = this position CANNOT be broken
```

```java
class Solution {
    Boolean[] memo;

    public boolean wordBreak(String s, List<String> wordDict) {
        memo = new Boolean[s.length()+1];
        return solve(s, wordDict, 0);
    }

    private boolean solve(String s, List<String> wordDict, int start) {
        if (start == s.length()) return true;
        if (memo[start] != null) return memo[start];  // return cached true OR false

        for (int i = start+1; i <= s.length(); i++) {
            if (wordDict.contains(s.substring(start, i))) {
                if (solve(s, wordDict, i)) {
                    return memo[start] = true;
                }
            }
        }

        return memo[start] = false;
    }
}
```

---

## Correct solution — tabulation

**Fill direction:** `solve(i)` depends on `solve(j)` where `j > i` — needs RIGHT → fill RIGHT TO LEFT.

Wait — actually `solve(start)` calls `solve(i)` where `i > start`. So needs values to the RIGHT already filled → fill RIGHT TO LEFT.

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        int n = s.length();
        boolean[] dp = new boolean[n+1];
        dp[n] = true;   // base case — empty suffix can always be "broken"

        for (int start = n-1; start >= 0; start--) {      // right to left
            for (int i = start+1; i <= n; i++) {
                if (dp[i] && wordDict.contains(s.substring(start, i))) {
                    dp[start] = true;
                    break;                                  // found one valid break
                }
            }
        }

        return dp[0];
    }
}
```

**Why `dp[i]` checked first:** `dp[i]` tells you if the suffix starting at `i` can be broken. If yes AND the word `s[start..i]` is in dictionary → `dp[start]` is true.

---

## Slot trace — s="leetcode", dict=["leet","code"]

```
dp = [F, F, F, F, F, F, F, F, T]   ← dp[8]=true (base case)

start=7: s[7..8]="e" → not in dict → dp[7]=false
start=6: s[6..8]="de" → not in dict → dp[6]=false
start=5: s[5..8]="ode" → not in dict → dp[5]=false
start=4: s[4..8]="code" → in dict, dp[8]=true → dp[4]=true ✅
start=3: s[3..4]="t", dp[4]=true? NO (dp[4]=true but "t" not in dict)
         s[3..8]="tcode" → not in dict → dp[3]=false
start=2: s[2..4]="et" → not in dict
         s[2..8]="etcode" → not in dict → dp[2]=false
start=1: s[1..4]="eet" → not in dict
         s[1..8]="eetcode" → not in dict → dp[1]=false
start=0: s[0..4]="leet" → in dict, dp[4]=true → dp[0]=true ✅

return dp[0] = true ✅
```

---

## 5 Questions Applied

```
1. What changes between calls?
   → start (current position in string)
   → memo[start], size n+1

2. Answer == return value?
   → YES — boolean IS what we return
   → track inside function

3. How many choices?
   → variable → loop over all possible end positions i

4. Base case meaning?
   → start == s.length() → empty suffix → return true
   → fully matched, nothing left to break

5. Fill direction?
   → solve(i) where i > start → needs RIGHT → fill RIGHT TO LEFT
   → base case: dp[n] = true
   → answer: dp[0]
```

---

## The boolean memo rule

```
answer is boolean? → use Boolean[] not boolean[]
                     null  = not computed yet
                     true  = computed, answer is true
                     false = computed, answer is false

answer is int?     → use int[] with sentinel
                     -1 = not computed (if -1 can't be real answer)
                     -2 = not computed (if -1 IS a valid answer, like coin change)
```

**Why `boolean[]` breaks:**
```
boolean[] memo = new boolean[n];   // all initialized to false
memo[start] = false                // computed false — same as uncomputed!
→ can't tell if false means "not computed" or "computed and answer is false"
→ always recomputes false positions — defeats the purpose of memo
```

---

## Why this is different from coin change loop

```
Coin change loop:  trying different COINS at same amount
                   → reduce amount, try all denominations

Word break loop:   trying different END POSITIONS for current word
                   → extend word until dictionary match found
```

Same mechanism (loop inside recursion), different purpose.

---

## One line to remember

> At each position, try every possible word ending here. If word is in dictionary AND rest of string is breakable → return true. Use Boolean[] not boolean[] for memo — three states needed.
