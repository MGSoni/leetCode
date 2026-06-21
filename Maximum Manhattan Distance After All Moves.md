# 3445. Maximum Manhattan Distance After All Moves

## Problem

You are given a string `moves` consisting of the characters `'U'`, `'D'`, `'L'`, `'R'`, and `'_'`.

Starting from the origin `(0, 0)`, each character represents one move on a 2D plane:

* `'U'`: Move up by 1 unit.
* `'D'`: Move down by 1 unit.
* `'L'`: Move left by 1 unit.
* `'R'`: Move right by 1 unit.
* `'_'`: Can be independently replaced with any one of `'U'`, `'D'`, `'L'`, or `'R'`.

Return the maximum Manhattan distance from the origin that can be achieved after all moves have been performed.

---

## Intuition

The final Manhattan distance is:

`|x| + |y|`

For the horizontal axis:

* `L` contributes `-1`
* `R` contributes `+1`

Final horizontal contribution:

`|L - R|`

For the vertical axis:

* `U` contributes `+1`
* `D` contributes `-1`

Final vertical contribution:

`|U - D|`

Each `'_'` can be assigned optimally to increase the Manhattan distance by exactly `1`.

Therefore, if the number of `'_'` characters is `k`, the maximum possible distance is:

`|L - R| + |U - D| + k`

---

## Approach

1. Count occurrences of:

   * `L`
   * `R`
   * `U`
   * `D`
   * `_`
2. Compute:

   * Horizontal distance = `|L - R|`
   * Vertical distance = `|U - D|`
3. Add the number of underscores since each can contribute `+1` to the final Manhattan distance.

---

## Complexity Analysis

* Time Complexity: `O(n)`
* Space Complexity: `O(1)`

---

## Java Solution

```java
class Solution {
    public int maxDistance(String moves) {
        int lCount = 0;
        int rCount = 0;
        int dCount = 0;
        int uCount = 0;
        int independentCount = 0;

        for (int i = 0; i < moves.length(); i++) {
            char ch = moves.charAt(i);

            if (ch == 'U') {
                uCount++;
            } else if (ch == 'D') {
                dCount++;
            } else if (ch == 'L') {
                lCount++;
            } else if (ch == 'R') {
                rCount++;
            } else {
                independentCount++;
            }
        }

        return Math.abs(lCount - rCount)
                + Math.abs(uCount - dCount)
                + independentCount;
    }
}
```
