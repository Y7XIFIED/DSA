# LeetCode #1096 - Brace Expansion II (Recursion, Hard)

## Problem Statement
Under the grammar given below, strings can represent a set of lowercase words. Let `R(expr)` denote the set of words the expression represents.

The grammar follows three fundamental rules:
1. **Single letters**: For every lowercase letter `x`, `R(x) = {x}`.
2. **Union**: For expressions `e1, e2, ... , ek` with `k >= 2`, `R({e1, e2, ...}) = R(e1) U R(e2) U ...`. (Represented by comma-separated expressions).
3. **Cartesian Product (Concatenation)**: For expressions `e1` and `e2`, `R(e1 + e2) = {a + b for (a, b) in R(e1) x R(e2)}`, where `+` denotes concatenation and `x` denotes the Cartesian product. (Represented by adjacent expressions).

Given an expression representing a set of words under the given grammar, return the sorted list of words that the expression represents without duplicates.

## Example

### Example 1
- **Input**: `expression = "{a,b}{c,{d,e}}"`
- **Output**: `["ac","ad","ae","bc","bd","be"]`
- **Explanation**: 
  - `{d,e}` expands to `{"d", "e"}`.
  - `{c,{d,e}}` is the union of `{"c"}` and `{"d", "e"}` -> `{"c", "d", "e"}`.
  - `{a,b}` expands to `{"a", "b"}`.
  - The concatenation `{a,b}{c,{d,e}}` is the Cartesian product of `{"a", "b"}` and `{"c", "d", "e"}`:
    `"a"` concatenated with each of `{"c", "d", "e"}` -> `"ac", "ad", "ae"`.
    `"b"` concatenated with each of `{"c", "d", "e"}` -> `"bc", "bd", "be"`.
  - Sorted output: `["ac","ad","ae","bc","bd","be"]`.

### Example 2
- **Input**: `expression = "{{a,z},a{b,c},{ab,z}}"`
- **Output**: `["a","ab","ac","z"]`
- **Explanation**: 
  - `{a,z}` -> `{"a", "z"}`.
  - `a{b,c}` -> `{"ab", "ac"}`.
  - `{ab,z}` -> `{"ab", "z"}`.
  - Taking the union gives `{"a", "ab", "ac", "z"}`. Each distinct word appears only once in the final answer.

## Approach
Expressions with nested braces and multiple operators (concatenation and union) resemble arithmetic expressions with operator precedence:
- **Union (`,`)** acts like addition (lower precedence).
- **Concatenation (adjacency)** acts like multiplication (higher precedence).
- **Braces (`{...}`)** act like parentheses (highest precedence, overriding default order).

Attempting string replacements or regex transformations quickly breaks down on deeply nested structures like `{{a,b},{c,{d,e}}}`. 

The ideal, industrial-strength technique to handle this grammar is a **Recursive Descent Parser**:
- `parseExpression()`: Handles union operations separated by commas (`,`).
- `parseTerm()`: Handles concatenation of adjacent factors (Cartesian product).
- `parseFactor()`: Handles atomic elements (single characters) or evaluates subexpressions enclosed within `{` and `}`.

This cleanly mirrors formal Context-Free Grammars (CFG) and evaluates expressions in a single structured sweep.

## Logic
- **Grammar Hierarchy**:
  - `Expression := Term (',' Term)*`
  - `Term := Factor+`
  - `Factor := Character | '{' Expression '}'`
- **Data Structures**:
  - `Set<String>` (via `HashSet`): Eliminates duplicate words automatically during unions and Cartesian products.
  - `List<String>`: Holds the deduplicated results, sorted lexicographically via `Collections.sort()`.
- **Parsing Flow**:
  - `parseExpression`: Evaluates the first `Term`, then continuously consumes commas (`,`) and takes the union (`addAll`) of subsequent terms.
  - `parseTerm`: Starts with an identity accumulator set containing the empty string `""`. It iteratively parses adjacent `Factor` instances and computes the Cartesian product between the accumulated set and the new factor set.
  - `parseFactor`: If the current character is `{`, it skips the open brace, recursively calls `parseExpression()`, and skips the closing `}`. Otherwise, it reads a single literal character into a singleton set.

## Algorithm
1. Store expression string `s` globally and initialize pointer `index = 0`.
2. Call `parseExpression()` to compute the root set of words:
   - Call `parseTerm()` to get the initial set.
   - While `s.charAt(index) == ','`:
     - Advance `index++`.
     - Evaluate `parseTerm()` and add all results to the union set.
   - Return the union set.
3. In `parseTerm()`:
   - Initialize accumulator `result = {""}`.
   - While `index` is within bounds and the character is neither `}` nor `,`:
     - Evaluate `next = parseFactor()`.
     - Form a new set containing `a + b` for each `a` in `result` and `b` in `next`.
     - Update `result` to this new Cartesian product set.
   - Return `result`.
4. In `parseFactor()`:
   - If `s.charAt(index) == '{'`:
     - Advance `index++`.
     - Evaluate `inner = parseExpression()`.
     - Advance `index++` (skip `}`).
     - Return `inner`.
   - Else:
     - Return a singleton set with `String.valueOf(s.charAt(index++))`.
5. Convert the final `Set<String>` into an `ArrayList`, sort it with `Collections.sort()`, and return the result.

## Java Solution

```java
import java.util.*;

class Solution {

    private String s;
    private int index;

    public List<String> braceExpansionII(String expression) {
        s = expression;
        index = 0;

        Set<String> result = parseExpression();

        List<String> answer = new ArrayList<>(result);
        Collections.sort(answer);

        return answer;
    }

    private Set<String> parseExpression() {
        Set<String> result = parseTerm();

        while (index < s.length() && s.charAt(index) == ',') {
            index++;
            result.addAll(parseTerm());
        }

        return result;
    }

    private Set<String> parseTerm() {
        Set<String> result = new HashSet<>();
        result.add("");

        while (index < s.length()
                && s.charAt(index) != '}'
                && s.charAt(index) != ',') {

            Set<String> next = parseFactor();
            Set<String> combined = new HashSet<>();

            for (String a : result) {
                for (String b : next) {
                    combined.add(a + b);
                }
            }

            result = combined;
        }

        return result;
    }

    private Set<String> parseFactor() {
        if (s.charAt(index) == '{') {
            index++;

            Set<String> result = parseExpression();

            index++;
            return result;
        }

        Set<String> result = new HashSet<>();
        result.add(String.valueOf(s.charAt(index)));

        index++;

        return result;
    }
}
```

## Dry Run

Tracing `expression = "{a,b}{c,{d,e}}"`:

| Parser Call | Current `index` | Lookahead Char | Action / Sub-call | Sub-Result Returned | Cumulative Result |
|---|---|---|---|---|---|
| `parseExpression` | `0` | `{` | Call `parseTerm` | - | - |
| `parseTerm` | `0` | `{` | `result = {""}`; call `parseFactor` | - | `result = {""}` |
| `parseFactor` | `0` | `{` | Skip `{`; call `parseExpression("a,b")` | `{"a", "b"}` | - |
| `parseTerm` | `5` | `{` | Product `result {""} x {"a", "b"}` | - | `result = {"a", "b"}` |
| `parseFactor` | `5` | `{` | Skip `{`; call `parseExpression("c,{d,e}")` | `{"c", "d", "e"}` | - |
| `parseTerm` | `16` | `EOF` | Product `{"a", "b"} x {"c", "d", "e"}` | `{"ac","ad","ae","bc","bd","be"}` | `{"ac","ad","ae","bc","bd","be"}` |

After `parseExpression` completes, the set is converted to a list and sorted:
`["ac", "ad", "ae", "bc", "bd", "be"]`.

## Complexity

- **Time Complexity**: $O(M \log M \times L + N)$
  - Let $N$ be the length of the expression ($N \le 60$), $M$ be the total number of words generated, and $L$ be the maximum word length.
  - The parsing logic visits each token in the expression string in $O(N)$ structural traversals.
  - Cartesian products and unions take time proportional to the number of combinations generated.
  - Sorting the final $M$ words takes $O(M \log M \times L)$ comparisons.
- **Space Complexity**: $O(M \times L + N)$
  - The recursion call stack uses at most $O(N)$ depth.
  - Intermediate hash sets and the final list store up to $M$ strings of length $L$.
  - Total auxiliary space is bounded by $O(M \times L + N)$.

## Key Concept
The key concept is **Recursive Descent Parsing for Operator Precedence Grammars**. Treating brace expressions as formal language grammars allows us to naturally map concatenation to multiplication (higher precedence) and comma-separated unions to addition (lower precedence). By structuring parsing into Expression, Term, and Factor, all associativity and precedence rules resolve automatically through the call stack.

## What I Learned
- Formal parsing hierarchies cleanly replace complicated iterative stack simulations when dealing with mixed operator precedences.
- Initializing the Cartesian product accumulator with an empty string `result = {""}` acts as the multiplicative identity element, keeping concatenation logic concise.
- Set operations inherently enforce uniqueness at every evaluation step, minimizing redundant work early.
