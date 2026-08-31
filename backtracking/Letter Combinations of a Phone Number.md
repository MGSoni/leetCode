# Letter Combinations of a Phone Number

**Pattern:** Recursion / Backtracking

## Problem
Given a string containing digits from `2-9`, return all possible letter combinations
that the number could represent, using the standard phone keypad mapping
(2→"abc", 3→"def", ..., 9→"wxyz").

Example: `digits = "23"` → `["ad","ae","af","bd","be","bf","cd","ce","cf"]`

## Approach
- Build a digit → letters map (`Character` → `String`, matching what `charAt()` returns).
- Recursively build one combination at a time using a shared `StringBuilder`:
  - **Base case:** when `currentString.length() == digits.length()`, the combination
    is complete — save a copy and return.
  - **Recursive step:** figure out which digit to work on using
    `currentString.length()` as the index into `digits` (it always equals how many
    letters have been placed so far, so no separate counter is needed).
    For each letter in that digit's letter-set:
    1. `append` the letter to `currentString`
    2. recurse
    3. `deleteCharAt` the last character (undo / backtrack) before trying the next letter
- **Edge case:** `digits == ""` must return `[]`, not `[""]`. Without an explicit guard,
  the base case triggers immediately on the empty string and adds an empty string to
  the result. Guard at the top of the public method:
  `if (digits.length() == 0) return new ArrayList<String>();`

## Complexity
- Time: O(4^n · n) worst case (digits like 7/9 map to 4 letters), where n = digits.length()
- Space: O(n) recursion depth, excluding the output list

## Solution

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        if (digits.length() == 0) return new ArrayList<String>();

        Map<Character, String> map = new HashMap<>();
        map.put('2',"abc");
        map.put('3',"def");
        map.put('4',"ghi");
        map.put('5',"jkl");
        map.put('6',"mno");
        map.put('7',"pqrs");
        map.put('8',"tuv");
        map.put('9',"wxyz");
        List<String> resultList = new ArrayList<>();
        StringBuilder currentString = new StringBuilder();

        function(digits, map, resultList, currentString);
        return resultList;
    }

    private void function(String digits, Map<Character, String> map, List<String> resultList, StringBuilder currentString) {

        if (currentString.length() == digits.length()) {
            resultList.add(currentString.toString());
            return;
        }

        String s = map.get(digits.charAt(currentString.length()));

        for (int i = 0; i < s.length(); i++) {
            currentString.append(s.charAt(i));
            function(digits, map, resultList, currentString);
            currentString.deleteCharAt(currentString.length() - 1);
        }
    }
}
```

## Mistakes made while solving

| What went wrong | Fix / insight |
|---|---|
| Initially reached for `Map<int, String>` — mismatched key type against what `charAt()` actually returns | Match the map's key type to what you naturally have in hand (`char`/`Character`, not `int`) to avoid unnecessary conversions |
| Missed the `digits.length() == 0` edge case initially — base case would've added an empty string to the result instead of returning an empty list | Always trace what your base case does when the input is empty/degenerate, before assuming the recursion "just handles it" |
