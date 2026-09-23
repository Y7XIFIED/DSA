# LeetCode #1 - Two Sum (Arrays, Easy)

## Problem Statement
Given an array of integers `nums` and an integer `target`, return the indices of the two numbers such that they add up to `target`.

You may assume that each input would have exactly one solution, and you may not use the same element twice. You can return the answer in any order.

## Example

### Example 1
- **Input**: `nums = [2, 7, 11, 15]`, `target = 9`
- **Output**: `[0, 1]`
- **Explanation**: Because `nums[0] + nums[1] == 2 + 7 == 9`, we return `[0, 1]`.

### Example 2
- **Input**: `nums = [3, 2, 4]`, `target = 6`
- **Output**: `[1, 2]`
- **Explanation**: Because `nums[1] + nums[2] == 2 + 4 == 6`, we return `[1, 2]`.

### Example 3
- **Input**: `nums = [3, 3]`, `target = 6`
- **Output**: `[0, 1]`
- **Explanation**: Because `nums[0] + nums[1] == 3 + 3 == 6`, we return `[0, 1]`.

## Approach
The naive approach is to use nested loops to test every possible pair:
For every index `i`, loop through every index `j > i` and check if `nums[i] + nums[j] == target`. While this works, checking all pairs takes quadratic time, giving an $O(n^2)$ time complexity. For large arrays, this becomes unacceptably slow.

To optimize, we change our perspective from searching forward to remembering what we have already seen. If two numbers sum to `target`:
$$x + y = \text{target} \implies y = \text{target} - x$$
For every number `nums[i]` we encounter, the exact number needed to complete the sum is its complement: `target - nums[i]`. Instead of scanning the remaining array repeatedly, we query a hash map in $O(1)$ average time to see if that complement was already visited earlier. If it was, we have found our answer in a single linear pass.

## Logic
- **Data Structure**: `HashMap<Integer, Integer>`
  - Key: Element value (`nums[i]`)
  - Value: Index of the element (`i`)
- **Single-Pass Lookup**: As we iterate across the array, we compute `complement = target - nums[i]`.
- **Condition Check**:
  - If `map.containsKey(complement)` is `true`, we immediately return `new int[] {map.get(complement), i}`.
  - If `false`, we record the current number and its index in the map (`map.put(nums[i], i)`) so future elements can reference it.
- **Handling Duplicates**: By looking up the complement before adding the current element, duplicate values that add up to target (such as `[3, 3]` with target `6`) are cleanly handled without collision or self-pairing.

## Algorithm
1. Initialize an empty hash map `map` where keys are integers and values are array indices.
2. Iterate through `nums` using index `i` from `0` to `nums.length - 1`.
3. At each index `i`, compute the required partner value: `complement = target - nums[i]`.
4. Check if `complement` exists in `map`:
   - If yes: Return an integer array containing `map.get(complement)` and `i`.
   - If no: Add the entry `(nums[i], i)` to `map`.
5. If the loop completes without returning, return an empty array `new int[] {}` (fallback).

## Java Solution

```java
import java.util.HashMap;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];

            if (map.containsKey(complement)) {
                return new int[] {map.get(complement), i};
            }

            map.put(nums[i], i);
        }

        return new int[] {};
    }
}
```

## Dry Run

Tracing with `nums = [2, 7, 11, 15]` and `target = 9`:

| Iteration `i` | Current `nums[i]` | Complement (`target - nums[i]`) | Map Check (`map.containsKey`) | Map State Before Put | Action Taken |
|---|---|---|---|---|---|
| `0` | `2` | `9 - 2 = 7` | `false` | `{}` | Put `(2, 0)` into map; proceed |
| `1` | `7` | `9 - 7 = 2` | `true` (index `0`) | `{2: 0}` | Complement found! Return `[0, 1]` |

The algorithm terminates at `i = 1` and returns `[0, 1]`.

## Complexity

- **Time Complexity**: $O(n)$
  - We traverse the list containing $n$ elements exactly once.
  - Each lookup and insertion in a hash table takes $O(1)$ time on average.
  - Total time: $n \times O(1) = O(n)$.
- **Space Complexity**: $O(n)$
  - In the worst case, the complement is found at the very last pair or we store up to $n$ elements in the hash map.
  - Therefore, auxiliary space scales linearly with input size: $O(n)$.

## Key Concept
The key concept here is **trading auxiliary space for faster time complexity** via a Hash Map. By spending $O(n)$ extra memory to remember previously observed values, we reduce the search query time from an $O(n)$ linear scan to an $O(1)$ constant-time lookup, dropping overall runtime from $O(n^2)$ down to $O(n)$.

## What I Learned
- Rather than running repeated nested searches forward, checking backwards against cached history fundamentally cuts down time complexity.
- Storing items dynamically in a hash map as we iterate prevents an element from matching with itself and cleanly handles duplicate numbers.
