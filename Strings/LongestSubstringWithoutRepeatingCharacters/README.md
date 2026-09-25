# LeetCode #3 - Longest Substring Without Repeating Characters (Strings, Medium)

## Problem Statement
Given a string `s`, find the length of the longest substring without duplicate characters.

A substring is a contiguous non-empty sequence of characters within a string.

## Example

### Example 1
- **Input**: `s = "abcabcbb"`
- **Output**: `3`
- **Explanation**: The answer is `"abc"`, with the length of `3`. Note that `"bca"` and `"cab"` are also valid substrings of length `3`.

### Example 2
- **Input**: `s = "bbbbb"`
- **Output**: `1`
- **Explanation**: The answer is `"b"`, with the length of `1`.

### Example 3
- **Input**: `s = "pwwkew"`
- **Output**: `3`
- **Explanation**: The answer is `"wke"`, with the length of `3`. Notice that the answer must be a substring; `"pwke"` is a subsequence and not a contiguous substring.

## Approach
A brute-force strategy would generate every possible substring, check if it contains all unique characters using a hash set, and keep track of the maximum length found. Because there are $O(n^2)$ total substrings and inspecting each substring of length up to $n$ requires $O(n)$ time, this naive approach runs in $O(n^3)$ time, easily timing out for strings of length $10^5$.

To achieve optimal performance, we apply the **Dynamic Sliding Window** pattern combined with a **Hash Map**:
1. We maintain a contiguous window defined by two pointers: `left` and `right`.
2. As the `right` pointer expands the window one character at a time, we inspect if the character was already encountered.
3. Instead of contracting the window slowly one index at a time, we jump the `left` pointer directly past the previous occurrence of the duplicate character.
4. Using `Math.max(left, lastSeenIndex + 1)` guarantees that `left` only moves forward and never moves backward into previously discarded characters.
5. This allows us to process the entire string in a single linear pass of $O(n)$ time.

## Logic
- **Data Structure**: `HashMap<Character, Integer>`
  - Key: The character (`currentChar`).
  - Value: The most recent index where this character was observed.
- **Pointers**:
  - `right`: Scans forward through the string from index `0` to `s.length() - 1`.
  - `left`: Marks the inclusive beginning of the current valid substring without duplicates.
- **Skip Duplicate via Hash Map**:
  - When `map.containsKey(currentChar)` is `true`, we find its previous index.
  - If the previous index falls within our current window (`>= left`), we shift `left = map.get(currentChar) + 1`.
  - Using `Math.max(left, map.get(currentChar) + 1)` cleanly avoids moving `left` backwards when encountering characters seen outside the current active window.
- **Window Length Tracking**:
  - The length of the current substring is `right - left + 1`.
  - We update `maxLength = Math.max(maxLength, right - left + 1)`.

## Algorithm
1. Initialize a hash map `map` to store the most recent index of each character.
2. Initialize integer variables `maxLength = 0` and `left = 0`.
3. Loop variable `right` from `0` to `s.length() - 1`:
   - Retrieve `currentChar = s.charAt(right)`.
   - If `currentChar` exists in `map`, update `left = Math.max(left, map.get(currentChar) + 1)`.
   - Insert or update `(currentChar, right)` into `map`.
   - Calculate current window size `right - left + 1` and update `maxLength = Math.max(maxLength, right - left + 1)`.
4. Return `maxLength`.

## Java Solution

```java
import java.util.HashMap;

class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer> map = new HashMap<>();
        int maxLength = 0;
        int left = 0;

        for (int right = 0; right < s.length(); right++) {
            char currentChar = s.charAt(right);

            if (map.containsKey(currentChar)) {
                left = Math.max(left, map.get(currentChar) + 1);
            }

            map.put(currentChar, right);
            maxLength = Math.max(maxLength, right - left + 1);
        }

        return maxLength;
    }
}
```

## Dry Run

Tracing with `s = "abcabcbb"`:

| `right` | `currentChar` | `map.containsKey` | `left` Calculation | Active Window `[left, right]` | Window Size (`right - left + 1`) | `maxLength` |
|---|---|---|---|---|---|---|
| `0` | `'a'` | `false` | `0` (unchanged) | `"a"` | `1` | `1` |
| `1` | `'b'` | `false` | `0` (unchanged) | `"ab"` | `2` | `2` |
| `2` | `'c'` | `false` | `0` (unchanged) | `"abc"` | `3` | `3` |
| `3` | `'a'` | `true` (index `0`) | `Math.max(0, 0 + 1) = 1` | `"bca"` | `3` | `3` |
| `4` | `'b'` | `true` (index `1`) | `Math.max(1, 1 + 1) = 2` | `"cab"` | `3` | `3` |
| `5` | `'c'` | `true` (index `2`) | `Math.max(2, 2 + 1) = 3` | `"abc"` | `3` | `3` |
| `6` | `'b'` | `true` (index `4`) | `Math.max(3, 4 + 1) = 5` | `"cb"` | `2` | `3` |
| `7` | `'b'` | `true` (index `6`) | `Math.max(5, 6 + 1) = 7` | `"b"` | `1` | `3` |

Final returned maximum length: `3`.

## Complexity

- **Time Complexity**: $O(n)$
  - The `right` pointer iterates across the string of length $n$ exactly once.
  - Hash map lookups and insertions execute in $O(1)$ average time.
  - The `left` pointer jumps directly without nested backtracking.
  - Overall time complexity is strictly linear: $O(n)$.
- **Space Complexity**: $O(\min(n, \Sigma))$
  - Auxiliary space is bounded by the size of the alphabet $\Sigma$ (character set) and the length of the string $n$.
  - For standard ASCII inputs, the map holds at most 128 distinct entries, effectively constant $O(1)$ space, or bounded by $O(\min(n, \Sigma))$ for general character sets.

## Key Concept
The key concept is the **Optimized Sliding Window with Direct Index Jumps**. While a basic sliding window with a set requires removing characters one by one until the duplicate is dropped (which could take $2n$ operations), storing the exact index of each character allows the left boundary to leap directly past duplicates in $O(1)$ time, delivering optimal single-pass execution.

## What I Learned
- `Math.max(left, map.get(currentChar) + 1)` is crucial to prevent the left pointer from moving backwards when encountering stale duplicate entries that fall outside the current window (e.g. in test cases like `"abba"`).
- Decoupling window expansion (`right++`) from smart boundary contraction gives an intuitive and performant model for substring problems.
