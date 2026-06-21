# 3614. Process String with Special Operations II

## Problem

You are given a string `s` consisting of lowercase English letters and the special characters: `'*'`, `'#'`, and `'%'`.

You are also given an integer `k`.

Build a new string `result` by processing `s` according to the following rules from left to right:

* If the letter is a lowercase English letter append it to `result`.
* A `'*'` removes the last character from `result`, if it exists.
* A `'#'` duplicates the current `result` and appends it to itself.
* A `'%'` reverses the current `result`.

Return the kth character of the final string `result`. If `k` is out of the bounds of `result`, return `'.'`.

### Example 1

Input:

```text
s = "a#b%*"
k = 1
```

Output:

```text
"a"
```

### Example 2

Input:

```text
s = "cd%#*#"
k = 3
```

Output:

```text
"d"
```

### Example 3

Input:

```text
s = "z*#"
k = 0
```

Output:

```text
"."
```

### Constraints

```text
1 <= s.length <= 10^5
0 <= k <= 10^15
Length of result <= 10^15
```

---

## Pattern Recognition

At first glance, the problem looks like a string simulation problem.

However:

```text
result length can be as large as 10^15
```

So actually building the final string is impossible.

We need to determine:

```text
Which original character eventually lands at index k?
```

This is a classic:

```text
Work backwards from the final state.
```

pattern.

---

## Key Observation

We do not need the entire string.

We only need:

```text
final length
and
the position k
```

If we know how every operation transforms the string length, we can reverse those transformations and keep mapping k back to its original position.

---

## Phase 1: Compute Final Length

Traverse from left to right and maintain only the length.

### Letter

```text
append one character
```

```java
len++
```

### *

```text
remove last character if present
```

```java
if(len > 0) len--
```

###

```text
duplicate string
```

```java
len *= 2
```

### %

```text
reverse string
```

Length remains unchanged.

After processing all operations:

```java
if(k >= len) return '.';
```

because k is outside the final string.

---

## Phase 2: Reverse the Operations

Now traverse from right to left.

At every step:

```text
Current len = length AFTER this operation
```

We undo the operation and update:

```text
k
len
```

accordingly.

---

## Reverse of Letter

Forward:

```text
append character
```

Example:

```text
abc -> abcd
```

Length:

```text
3 -> 4
```

While reversing:

If:

```text
k == len - 1
```

then k points exactly to the appended character.

Therefore:

```java
if(k == len - 1) return ch;
```

Otherwise:

```java
len--;
```

and continue.

---

## Reverse of %

Forward:

```text
abcde -> edcba
```

Index mapping:

```text
0 -> 4
1 -> 3
2 -> 2
3 -> 1
4 -> 0
```

General formula:

```text
newIndex = len - 1 - oldIndex
```

Reversing gives:

```text
oldIndex = len - 1 - newIndex
```

Therefore:

```java
k = len - 1 - k;
```

Equivalent to:

```java
k = len - (k + 1);
```

---

## Reverse of *

Forward:

```text
remove last character
```

Example:

```text
abcd -> abc
```

Length:

```text
4 -> 3
```

When reversing:

```text
abc -> abcd
```

Only the length changes.

Index k remains valid because the deleted character was at the end.

Therefore:

```java
len++;
```

---

## Reverse of

Forward:

```text
abc -> abcabc
```

Length:

```text
3 -> 6
```

Let:

```java
oldLen = len / 2;
```

Current string:

```text
[first half][second half]
```

Both halves are identical.

If:

```java
k >= oldLen
```

then k lies in the duplicated half.

Map it back:

```java
k -= oldLen;
```

Then restore:

```java
len = oldLen;
```

---

## Example Walkthrough

```text
s = "a#b%*"
k = 1
```

Forward length computation:

```text
a  -> 1
#  -> 2
b  -> 3
%  -> 3
*  -> 2
```

Final length:

```text
2
```

k is valid.

Reverse traversal:

### *

```text
len = 2 -> 3
```

### %

```text
k = 3 - (1 + 1)
  = 1
```

### b

```text
k != len - 1
1 != 2

len = 2
```

###

```text
oldLen = 1

k >= 1

k = 0
len = 1
```

### a

```text
k == len - 1

0 == 0
```

Return:

```text
'a'
```

---

## Why This Works

Instead of constructing a string of size:

```text
10^15
```

we only maintain:

```text
len
k
```

and repeatedly ask:

```text
Where did this position come from before the current operation?
```

Eventually we reach the original character responsible for index k.

---

## Complexity

### Time

```text
O(n)
```

Two passes over the string.

### Space

```text
O(1)
```

Only a few variables are used.

---

## Java Solution

```java
class Solution {
    public char processStr(String s, long k) {

        long len = 0;

        for (int i = 0; i < s.length(); i++) {

            char ch = s.charAt(i);

            if (ch >= 'a' && ch <= 'z') {
                len++;
            }
            else if (ch == '*' && len > 0) {
                len--;
            }
            else if (ch == '#') {
                len *= 2;
            }
        }

        if (k >= len) {
            return '.';
        }

        for (int i = s.length() - 1; i >= 0; i--) {

            char ch = s.charAt(i);

            if (ch >= 'a' && ch <= 'z') {

                if (k == len - 1) {
                    return ch;
                }

                len--;
            }
            else if (ch == '%') {

                k = len - (k + 1);

            }
            else if (ch == '*') {

                len++;

            }
            else if (ch == '#') {

                long oldLen = len / 2;

                if (k >= oldLen) {
                    k -= oldLen;
                }

                len = oldLen;
            }
        }

        return '.';
    }
}
```
