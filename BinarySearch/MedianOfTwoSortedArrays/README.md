# LeetCode #4 - Median of Two Sorted Arrays (BinarySearch, Hard)

## Problem Statement
Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return the median of the two sorted arrays.

The overall run time complexity must be $O(\log(m + n))$.

## Example

### Example 1
- **Input**: `nums1 = [1, 3]`, `nums2 = [2]`
- **Output**: `2.00000`
- **Explanation**: Merged array = `[1, 2, 3]` and the median element is `2`.

### Example 2
- **Input**: `nums1 = [1, 2]`, `nums2 = [3, 4]`
- **Output**: `2.50000`
- **Explanation**: Merged array = `[1, 2, 3, 4]` and the median is `(2 + 3) / 2 = 2.5`.

## Approach
A straightforward method is to merge the two sorted arrays using the two-pointer merge routine from Merge Sort and then pick the middle element. However, merging takes $O(m + n)$ time and $O(m + n)$ space (or $O(1)$ extra space by counting up to the median), which violates the strict $O(\log(m + n))$ requirement.

To reach logarithmic time, we do not need to construct the merged array. Instead, we use **Binary Search on the Partition Point**:
1. The median divides the combined elements into two equal-sized left and right halves.
2. If we know how many elements from `nums1` belong to the left half (denoted by `partition1`), then the number of elements from `nums2` in the left half is automatically fixed: `partition2 = (m + n + 1) / 2 - partition1`.
3. A partition is correct when every element in the combined left half is less than or equal to every element in the combined right half. Because both individual arrays are already sorted, we only need to verify two boundary cross-conditions:
   - `maxLeft1 <= minRight2`
   - `maxLeft2 <= minRight1`
4. If `maxLeft1 > minRight2`, we took too many elements from `nums1`, so we move our binary search range to the left. Otherwise, we move it to the right.
5. By always running the binary search on the shorter array, we ensure non-negative partition indices and achieve $O(\log(\min(m, n)))$ runtime.

## Logic
- **Search Space Reduction**:
  - If `nums1.length > nums2.length`, swap arguments: `findMedianSortedArrays(nums2, nums1)`.
  - Binary search boundaries: `left = 0`, `right = m`.
- **Partition Arithmetic**:
  - `partition1 = (left + right) / 2`
  - `partition2 = (m + n + 1) / 2 - partition1` (using `+ 1` ensures the left half has one extra element for odd total lengths).
- **Boundary Handling via Sentinel Values**:
  - If a partition cut is at index `0` (no elements on the left), we treat `maxLeft` as `Integer.MIN_VALUE`.
  - If a partition cut is at the array length (no elements on the right), we treat `minRight` as `Integer.MAX_VALUE`.
- **Median Calculation**:
  - Odd total length: `Math.max(maxLeft1, maxLeft2)`
  - Even total length: `(Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0`

## Algorithm
1. Check if `nums1.length > nums2.length`. If so, recursively call with swapped arrays so `nums1` is always the smaller array of size `m`.
2. Initialize binary search pointers `left = 0` and `right = m`.
3. Loop while `left <= right`:
   - Calculate cut points:
     - `partition1 = (left + right) / 2`
     - `partition2 = (m + n + 1) / 2 - partition1`
   - Determine the 4 edge values (`maxLeft1`, `minRight1`, `maxLeft2`, `minRight2`) using `Integer.MIN_VALUE` and `Integer.MAX_VALUE` for boundary cuts.
   - If `maxLeft1 <= minRight2` and `maxLeft2 <= minRight1`:
     - If `(m + n)` is odd, return `Math.max(maxLeft1, maxLeft2)`.
     - Else, return `(Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0`.
   - Else if `maxLeft1 > minRight2`:
     - Cut in `nums1` is too far right; update `right = partition1 - 1`.
   - Else:
     - Cut in `nums1` is too far left; update `left = partition1 + 1`.
4. Return `0.0` as fallback.

## Java Solution

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {

        // Always binary search on the smaller array
        if (nums1.length > nums2.length) {
            return findMedianSortedArrays(nums2, nums1);
        }

        int m = nums1.length;
        int n = nums2.length;

        int left = 0;
        int right = m;

        while (left <= right) {

            int partition1 = (left + right) / 2;
            int partition2 = (m + n + 1) / 2 - partition1;

            int maxLeft1 = (partition1 == 0)
                    ? Integer.MIN_VALUE
                    : nums1[partition1 - 1];

            int minRight1 = (partition1 == m)
                    ? Integer.MAX_VALUE
                    : nums1[partition1];

            int maxLeft2 = (partition2 == 0)
                    ? Integer.MIN_VALUE
                    : nums2[partition2 - 1];

            int minRight2 = (partition2 == n)
                    ? Integer.MAX_VALUE
                    : nums2[partition2];

            if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {

                if ((m + n) % 2 == 1) {
                    return Math.max(maxLeft1, maxLeft2);
                }

                return (Math.max(maxLeft1, maxLeft2)
                        + Math.min(minRight1, minRight2)) / 2.0;
            }

            if (maxLeft1 > minRight2) {
                right = partition1 - 1;
            } else {
                left = partition1 + 1;
            }
        }

        return 0.0;
    }
}
```

## Dry Run

Tracing `nums1 = [1, 2]` and `nums2 = [3, 4]`:
- Sizes: `m = 2`, `n = 2`. Total = `4` (even).
- Initial search range: `left = 0`, `right = 2`.

| Iteration | `left` | `right` | `partition1` | `partition2` | `maxLeft1` | `minRight1` | `maxLeft2` | `minRight2` | Condition Valid? | Action Taken |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | `0` | `2` | `1` | `1` | `1` | `2` | `3` | `4` | `maxLeft2 (3) <= minRight1 (2)` is `false` | `left = partition1 + 1 = 2` |
| 2 | `2` | `2` | `2` | `0` | `2` | `+INF` | `-INF` | `3` | `2 <= 3` and `-INF <= +INF` is `true` | Valid partition found! |

Computing the median for even length:
- `maxLeft = Math.max(2, -INF) = 2`
- `minRight = Math.min(+INF, 3) = 3`
- `Median = (2 + 3) / 2.0 = 2.5`

## Complexity

- **Time Complexity**: $O(\log(\min(m, n)))$
  - Binary search is conducted exclusively over the smaller array of size $m$.
  - The search space halves at every step: $m \to m/2 \to m/4 \dots$
  - Each iteration involves basic arithmetic comparisons in $O(1)$ time.
  - Overall time complexity is $O(\log(\min(m, n)))$, comfortably exceeding the problem requirement of $O(\log(m + n))$.
- **Space Complexity**: $O(1)$
  - Only a fixed number of pointer variables and sentinel boundaries are allocated.
  - Zero extra array allocations or recursive frame accumulations.

## Key Concept
The key concept is **Virtual Merging via Binary Search on Cut Positions**. Instead of finding the median element directly, we search for the correct divider between the left and right halves across two sorted collections. The property that elements are pre-sorted allows us to eliminate half of the candidate cut positions in each iteration.

## What I Learned
- Swapping inputs so that the first array is always smaller avoids negative index calculations in `partition2` and minimizes binary search iterations.
- Sentinel values (`Integer.MIN_VALUE` and `Integer.MAX_VALUE`) eliminate cumbersome edge-case checks when partition cuts land on array boundaries.
