# LeetCode #1807 - Evaluate the Bracket Pairs of a String (Hashing, Medium)

## Problem Statement
You are given a string `s` that contains some bracket pairs, with each pair containing a non-empty key.
For example, in the string `"(name)is(age)yearsold"`, there are two bracket pairs containing keys `"name"` and `"age"`.

You are given a list of key-value pairs `knowledge` where `knowledge[i] = [keyi, valuei]`.

Evaluate all bracket pairs according to these rules:
1. Replace `(keyi)` with its corresponding `valuei`.
2. If `keyi` is not present in `knowledge`, replace `(keyi)` with a question mark `"?"`.
3. Each key appears at most once in `knowledge`.
4. There are no nested brackets in `s`.

Return the evaluated resulting string.

## Example

### Example 1
- **Input**: `s = "(name)is(age)yearsold"`, `knowledge = [["name","bob"],["age","two"]]`
- **Output**: `"bobistwoyearsold"`
- **Explanation**: 
  - `(name)` is replaced with `"bob"`.
  - `(age)` is replaced with `"two"`.
  - Result: `"bobistwoyearsold"`.

### Example 2
- **Input**: `s = "hi(name)"`, `knowledge = [["a","b"]]`
- **Output**: `"hi?"`
- **Explanation**: The key `"name"` is not present in `knowledge`, so `(name)` is replaced with `"?"`.

### Example 3
- **Input**: `s = "(a)(a)(a)aaa"`, `knowledge = [["a","yes"]]`
- **Output**: `"yesyesyesaaa"`
- **Explanation**: The key `"a"` appears three times inside brackets and is replaced with `"yes"`. Literal `"a"`s outside brackets are kept as-is.

## Approach
A naive string replacement approach calling `s.replace("(key)", value)` for every entry in `knowledge` is inefficient because each string replacement scans the whole string from scratch. If `knowledge` contains $K$ keys and the string has length $N$, this would result in $O(K \times N)$ time complexity, leading to severe slowdowns on inputs of length $10^5$.

Instead, we solve this in optimal linear time using a **Hash Map Dictionary + Single-Pass Two-Pointer Parser**:
1. Preprocess all `[key, value]` pairs into a `HashMap<String, String>` for $O(1)$ average lookup time.
2. Traverse string `s` with a single index pointer `i`.
3. If `s.charAt(i)` is an ordinary character, append it directly to a `StringBuilder`.
4. When encountering an open bracket `'('`, scan forward to find the matching closing bracket `')'`.
5. Extract the substring between `i + 1` and `j` as the `key`.
6. Look up `key` in the hash map: if present, append the mapped value; otherwise, append `'?'`.
7. Move `i` directly to `j` and resume scanning, guaranteeing that each character is inspected only once.

## Logic
- **Data Structures**:
  - `HashMap<String, String>`: Provides $O(1)$ average time lookups for each bracketed key.
  - `StringBuilder`: Enables amortized $O(1)$ character and substring appends, avoiding immutable string concatenation overhead.
- **Parsing Invariants**:
  - Since brackets are strictly non-nested and valid, the characters between index `i` (where `s.charAt(i) == '('`) and `j` (where `s.charAt(j) == ')'`) form the exact key identifier.
  - Advancing `i = j` at the end of the bracket block skips reprocessing the enclosed key, maintaining strict $O(N)$ linear traversal.

## Algorithm
1. Populate a hash map `map` from the `knowledge` list mapping each `pair.get(0)` to `pair.get(1)`.
2. Initialize a `StringBuilder result`.
3. Loop through index `i` from `0` to `s.length() - 1`:
   - If `s.charAt(i) == '('`:
     - Find the matching closing bracket by advancing `j = i + 1` until `s.charAt(j) == ')'`.
     - Extract `key = s.substring(i + 1, j)`.
     - If `map.containsKey(key)`, append `map.get(key)` to `result`; otherwise append `'?'`.
     - Set `i = j` to skip past the evaluated pair.
   - Else:
     - Append `s.charAt(i)` directly to `result`.
4. Return `result.toString()`.

## Java Solution

```java
import java.util.*;

class Solution {
    public String evaluate(String s, List<List<String>> knowledge) {

        HashMap<String, String> map = new HashMap<>();

        for (List<String> pair : knowledge) {
            map.put(pair.get(0), pair.get(1));
        }

        StringBuilder result = new StringBuilder();

        for (int i = 0; i < s.length(); i++) {

            if (s.charAt(i) == '(') {
                int j = i + 1;

                while (s.charAt(j) != ')') {
                    j++;
                }

                String key = s.substring(i + 1, j);

                if (map.containsKey(key)) {
                    result.append(map.get(key));
                } else {
                    result.append('?');
                }

                i = j;
            } else {
                result.append(s.charAt(i));
            }
        }

        return result.toString();
    }
}
```

## Dry Run

Tracing `s = "(name)is(age)yearsold"`, `knowledge = [["name","bob"],["age","two"]]`:
- Map initialized: `{"name": "bob", "age": "two"}`

| Step `i` | Character / State | Substring Extracted (`key`) | Map Lookup Result | `result` State | Pointer Advance |
|---|---|---|---|---|---|
| `0` | `'('` | `"name"` (indices `1` to `4`) | `"bob"` | `"bob"` | `i = 5` (`')'`) |
| `6` | `'i'` | - | - | `"bobi"` | `i = 6` |
| `7` | `'s'` | - | - | `"bobis"` | `i = 7` |
| `8` | `'('` | `"age"` (indices `9` to `11`) | `"two"` | `"bobistwo"` | `i = 12` (`')'`) |
| `13..20` | `"yearsold"` | - | - | `"bobistwoyearsold"` | Normal character loop |

Final returned string: `"bobistwoyearsold"`.

## Complexity

- **Time Complexity**: $O(N + M)$
  - Let $N$ be the length of string `s` and $M$ be the total length of all keys and values in `knowledge`.
  - Building the hash map takes $O(M)$ time.
  - Scanning `s` visits each index at most twice (once to find `')'` and once in outer pointer movement), taking $O(N)$ time.
  - Overall time complexity is strictly linear: $O(N + M)$.
- **Space Complexity**: $O(M + N)$
  - The `HashMap` stores $K$ key-value pairs totaling $O(M)$ space.
  - `StringBuilder result` stores the constructed string of length up to $O(N + M)$.
  - Total auxiliary space is $O(M + N)$.

## Key Concept
The key concept is **Single-Pass Parser with $O(1)$ Hash Map Substitution**. By streaming through the characters once and parsing bracket delimiters locally, template evaluation scales linearly with input size and completely avoids repeated scans.

## What I Learned
- Building an inverted index or hash map upfront transforms what could be a repetitive $O(K \times N)$ string replace workflow into a clean $O(N + M)$ streaming pipeline.
- Synchronizing outer loop index jumps (`i = j`) directly skips the enclosed token body, preventing redundant string scans.
