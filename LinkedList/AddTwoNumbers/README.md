# LeetCode #2 - Add Two Numbers (LinkedList, Medium)

## Problem Statement
You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

## Example

### Example 1
- **Input**: `l1 = [2, 4, 3]`, `l2 = [5, 6, 4]`
- **Output**: `[7, 0, 8]`
- **Explanation**: The numbers represented are `342` and `465`. Their sum is `342 + 465 = 807`, represented in reverse as `[7, 0, 8]`.

### Example 2
- **Input**: `l1 = [0]`, `l2 = [0]`
- **Output**: `[0]`
- **Explanation**: `0 + 0 = 0`.

### Example 3
- **Input**: `l1 = [9, 9, 9, 9, 9, 9, 9]`, `l2 = [9, 9, 9, 9]`
- **Output**: `[8, 9, 9, 9, 0, 0, 0, 1]`
- **Explanation**: `9999999 + 9999 = 10009998`, which in reverse is `[8, 9, 9, 9, 0, 0, 0, 1]`.

## Approach
A naive idea might be to convert both linked lists into integers, add the numbers mathematically, and convert the result back into a new linked list. However, this approach fails because linked lists can contain up to 100 digits, exceeding the maximum range of primitive numeric types like 64-bit `long` or even causing memory overflow.

Instead, we take advantage of how the digits are stored: in reverse order. This matches elementary school addition where we sum digits from right to left (least significant to most significant), propagating a carry value to the next column. By traversing both linked lists simultaneously node by node, we can perform column-by-column addition in a single linear pass without ever worrying about integer overflow.

## Logic
- **Dummy Head Pointer**: We use a `dummyHead` node to simplify list construction. It provides a fixed reference point to the start of the result list and avoids messy null checks when inserting the first element.
- **Carry Tracking**: At each step, the total sum is initialized with the leftover `carry` from the previous position.
- **Variable Length Lists**: If one list is shorter than the other, we treat the missing nodes as having a value of `0` and continue processing until both lists are exhausted.
- **Remaining Carry**: After reaching the end of both lists, if `carry` is still greater than `0`, an additional node with the carry value must be appended.
- **Digit Extraction**:
  - Node value: `sum % 10`
  - Next carry: `sum / 10`

## Algorithm
1. Create a `dummyHead` node initialized with value `0`, and set `current` pointer to `dummyHead`.
2. Initialize integer `carry = 0`.
3. Loop while `l1 != null`, `l2 != null`, or `carry != 0`:
   - Initialize `sum = carry`.
   - If `l1` is not null, add `l1.val` to `sum` and advance `l1 = l1.next`.
   - If `l2` is not null, add `l2.val` to `sum` and advance `l2 = l2.next`.
   - Update `carry = sum / 10`.
   - Create a new node with value `sum % 10` and link it: `current.next = new ListNode(sum % 10)`.
   - Advance `current = current.next`.
4. Return `dummyHead.next` as the head of the resulting linked list.

## Java Solution

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummyHead = new ListNode(0);
        ListNode current = dummyHead;
        int carry = 0;

        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry;

            if (l1 != null) {
                sum += l1.val;
                l1 = l1.next;
            }

            if (l2 != null) {
                sum += l2.val;
                l2 = l2.next;
            }

            carry = sum / 10;
            current.next = new ListNode(sum % 10);
            current = current.next;
        }

        return dummyHead.next;
    }
}
```

## Dry Run

Tracing with `l1 = [2, 4, 3]` and `l2 = [5, 6, 4]`:

| Step | `l1.val` | `l2.val` | Input `carry` | `sum = carry + l1 + l2` | New Node (`sum % 10`) | Output `carry` (`sum / 10`) | Result List Built |
|---|---|---|---|---|---|---|---|
| Initial | - | - | `0` | - | - | `0` | `dummyHead(0)` |
| 1 | `2` | `5` | `0` | `0 + 2 + 5 = 7` | `7` | `0` | `(0) -> [7]` |
| 2 | `4` | `6` | `0` | `0 + 4 + 6 = 10` | `0` | `1` | `(0) -> [7] -> [0]` |
| 3 | `3` | `4` | `1` | `1 + 3 + 4 = 8` | `8` | `0` | `(0) -> [7] -> [0] -> [8]` |
| End | `null` | `null` | `0` | Loop terminates | - | - | Return `dummyHead.next`: `[7, 0, 8]` |

## Complexity

- **Time Complexity**: $O(\max(m, n))$
  - Let $m$ be the number of nodes in `l1` and $n$ be the number of nodes in `l2`.
  - The while loop executes at most $\max(m, n) + 1$ times to handle all digits and any potential final carry.
  - Each step does constant $O(1)$ arithmetic and pointer updates.
  - Overall time complexity is strictly linear: $O(\max(m, n))$.
- **Space Complexity**: $O(\max(m, n))$
  - Auxiliary space is utilized solely for creating the output linked list.
  - The length of the new list is at most $\max(m, n) + 1$.
  - Therefore, auxiliary space complexity is $O(\max(m, n))$.

## Key Concept
The key concept is **Elementary Addition with Dummy Node Management**. The reverse-order representation in a singly linked list allows us to mimic standard column-based addition without reversing lists or holding gigantic numbers in memory. Employing a dummy head node eliminates edge cases for list initialization, yielding elegant and bug-free pointer code.

## What I Learned
- Storing digits in reverse order is ideal for addition since addition naturally progresses from least significant to most significant digit.
- Combining list traversal conditions and carry checks into a single loop condition (`l1 != null || l2 != null || carry != 0`) keeps the logic unified and removes redundant cleanup loops.
- Dummy head pointers prevent conditional null-checking during linked list creation.
