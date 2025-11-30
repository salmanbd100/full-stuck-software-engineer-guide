# In-place Reversal of LinkedList Pattern

## Pattern Overview

The **In-place Reversal of LinkedList** pattern involves reversing a linked list or parts of it without using extra space. This is done by manipulating the `next` pointers of the nodes to change the direction of the links.

### When to Use
- Reversing an entire linked list
- Reversing a portion of a linked list (between positions m and n)
- Reversing nodes in k-group
- Reversing alternate k nodes
- Problems requiring pointer manipulation in linked lists

### Key Characteristics
- Modifies links in-place without creating new nodes
- Uses O(1) space (only pointer variables)
- Requires careful tracking of previous, current, and next nodes
- Changes the direction of `next` pointers

### Pattern Identification
Look for this pattern when you see:
- "Reverse a linked list"
- "Reverse nodes in k-group"
- "Reverse between positions m and n"
- "Swap nodes in pairs"
- Any problem involving linked list reversal

---

## Example 1: Reverse Linked List (JavaScript)

### Problem
Given the head of a singly linked list, reverse the list and return the reversed list.

**LeetCode**: [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

### Solution

```javascript
/**
 * Definition for singly-linked list node
 */
class ListNode {
    constructor(val, next = null) {
        this.val = val;
        this.next = next;
    }
}

/**
 * Reverse a linked list in-place (iterative approach)
 * @param {ListNode} head - Head of the linked list
 * @return {ListNode} - Head of the reversed linked list
 */
function reverseList(head) {
    // prev will eventually become the new head
    let prev = null;
    let current = head;

    while (current !== null) {
        // Store next node before changing the link
        const nextTemp = current.next;

        // Reverse the link: current node points to previous node
        current.next = prev;

        // Move prev and current one step forward
        prev = current;
        current = nextTemp;
    }

    // prev is now the new head of the reversed list
    return prev;
}

/**
 * Reverse a linked list using recursion
 * @param {ListNode} head
 * @return {ListNode}
 */
function reverseListRecursive(head) {
    // Base case: empty list or single node
    if (head === null || head.next === null) {
        return head;
    }

    // Recursively reverse the rest of the list
    const newHead = reverseListRecursive(head.next);

    // Reverse the link between current and next node
    head.next.next = head;
    head.next = null;

    return newHead;
}

// Helper function to create linked list from array
function createLinkedList(arr) {
    if (arr.length === 0) return null;

    const head = new ListNode(arr[0]);
    let current = head;

    for (let i = 1; i < arr.length; i++) {
        current.next = new ListNode(arr[i]);
        current = current.next;
    }

    return head;
}

// Helper function to print linked list
function printList(head) {
    const result = [];
    let current = head;

    while (current !== null) {
        result.push(current.val);
        current = current.next;
    }

    return result;
}

// Example usage
const head1 = createLinkedList([1, 2, 3, 4, 5]);
console.log("Original:", printList(head1));        // [1, 2, 3, 4, 5]

const reversed1 = reverseList(head1);
console.log("Reversed:", printList(reversed1));    // [5, 4, 3, 2, 1]

const head2 = createLinkedList([1, 2]);
const reversed2 = reverseListRecursive(head2);
console.log("Reversed:", printList(reversed2));    // [2, 1]
```

### Explanation (Iterative)

**Visual Step-by-Step** for list `1 → 2 → 3 → null`:

```
Initial:    prev = null, current = 1 → 2 → 3 → null

Step 1:
  nextTemp = 2 → 3 → null
  Reverse link: 1 → null
  Move: prev = 1, current = 2 → 3 → null
  Result: null ← 1    2 → 3 → null

Step 2:
  nextTemp = 3 → null
  Reverse link: 2 → 1
  Move: prev = 2, current = 3 → null
  Result: null ← 1 ← 2    3 → null

Step 3:
  nextTemp = null
  Reverse link: 3 → 2
  Move: prev = 3, current = null
  Result: null ← 1 ← 2 ← 3

Final: Return prev (which is 3)
```

**Key Points**:
1. **Three pointers**: `prev`, `current`, `nextTemp`
2. **Save next**: Always save `next` before changing links
3. **Reverse link**: Point `current.next` to `prev`
4. **Move forward**: Advance both `prev` and `current`

---

## Example 2: Reverse Linked List II (Python)

### Problem
Given the head of a singly linked list and two integers `left` and `right` where `left <= right`, reverse the nodes of the list from position `left` to position `right`, and return the reversed list.

**LeetCode**: [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)

### Solution

```python
from typing import Optional

class ListNode:
    """Definition for singly-linked list node"""
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        """
        Reverse nodes from position left to right (1-indexed)

        Args:
            head: Head of the linked list
            left: Starting position (1-indexed)
            right: Ending position (1-indexed)

        Returns:
            Head of the modified linked list
        """
        # Edge case: if left == right, no reversal needed
        if not head or left == right:
            return head

        # Create a dummy node to handle edge case where left = 1
        dummy = ListNode(0)
        dummy.next = head

        # Step 1: Move to the node just before the reversal point
        prev = dummy
        for _ in range(left - 1):
            prev = prev.next

        # Step 2: Reverse the sublist from left to right
        # prev is now the node before position 'left'
        # current is the first node to be reversed
        current = prev.next

        # Reverse the sublist
        for _ in range(right - left):
            # Save the next node
            next_node = current.next

            # Remove next_node from its current position
            current.next = next_node.next

            # Insert next_node at the beginning of the reversed portion
            next_node.next = prev.next
            prev.next = next_node

        return dummy.next

# Helper functions
def create_linked_list(values):
    """Create linked list from list of values"""
    if not values:
        return None

    head = ListNode(values[0])
    current = head

    for val in values[1:]:
        current.next = ListNode(val)
        current = current.next

    return head

def list_to_array(head):
    """Convert linked list to array for easy viewing"""
    result = []
    current = head

    while current:
        result.append(current.val)
        current = current.next

    return result

# Example usage
solution = Solution()

# Example 1: Reverse from position 2 to 4
head1 = create_linked_list([1, 2, 3, 4, 5])
result1 = solution.reverseBetween(head1, 2, 4)
print(list_to_array(result1))  # Output: [1, 4, 3, 2, 5]
# Explanation: Reversed portion [2,3,4] becomes [4,3,2]

# Example 2: Reverse single node
head2 = create_linked_list([5])
result2 = solution.reverseBetween(head2, 1, 1)
print(list_to_array(result2))  # Output: [5]

# Example 3: Reverse from beginning
head3 = create_linked_list([3, 5, 7, 9, 11])
result3 = solution.reverseBetween(head3, 1, 3)
print(list_to_array(result3))  # Output: [7, 5, 3, 9, 11]

# Example 4: Reverse to end
head4 = create_linked_list([1, 2, 3, 4, 5])
result4 = solution.reverseBetween(head4, 3, 5)
print(list_to_array(result4))  # Output: [1, 2, 5, 4, 3]
```

### Explanation

**Visual Step-by-Step** for reversing positions 2-4 in `1 → 2 → 3 → 4 → 5`:

```
Initial: 1 → 2 → 3 → 4 → 5
         ^
        prev (after moving left-1 times)

current = 2 (first node to reverse)

Iteration 1 (reverse 2 and 3):
  next_node = 3
  current.next = 4  (skip 3)
  next_node.next = 2  (3 points to 2)
  prev.next = 3  (1 points to 3)
  Result: 1 → 3 → 2 → 4 → 5

Iteration 2 (reverse 2 and 4):
  next_node = 4
  current.next = 5  (skip 4)
  next_node.next = 3  (4 points to 3)
  prev.next = 4  (1 points to 4)
  Result: 1 → 4 → 3 → 2 → 5

Final: 1 → 4 → 3 → 2 → 5
```

**Key Points**:
1. **Dummy node**: Handles edge case where left = 1
2. **Position prev**: Move to node just before reversal starts
3. **Iterative reversal**: Reverse `right - left` times
4. **Reattach**: The algorithm maintains connections to nodes outside the reversal range

---

## Time & Space Complexity

### Example 1: Reverse Linked List
- **Time Complexity**: O(n) - Visit each node once
- **Space Complexity**:
  - Iterative: O(1) - Only pointer variables
  - Recursive: O(n) - Call stack depth

### Example 2: Reverse Between Positions
- **Time Complexity**: O(n) - May traverse entire list in worst case
- **Space Complexity**: O(1) - Only pointer variables

---

## Common Variations

1. **Reverse Entire List**
   - Reverse complete linked list
   - LeetCode: [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

2. **Reverse Sublist**
   - Reverse nodes from position m to n
   - LeetCode: [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)

3. **Reverse Nodes in k-Group**
   - Reverse nodes in groups of k
   - LeetCode: [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

4. **Swap Nodes in Pairs**
   - Swap every two adjacent nodes
   - LeetCode: [24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)

5. **Reverse Alternate k Nodes**
   - Reverse every alternate k nodes
   - Similar to k-Group but skip alternate groups

---

## Practice Problems

### Easy
1. [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)
2. [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) (uses reversal)

### Medium
3. [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)
4. [24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)
5. [61. Rotate List](https://leetcode.com/problems/rotate-list/)
6. [143. Reorder List](https://leetcode.com/problems/reorder-list/) (uses reversal)

### Hard
7. [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

---

## Key Takeaways

1. **Three pointer technique**: `prev`, `current`, `next` are essential
2. **Save before change**: Always save `next` pointer before reversing links
3. **Dummy node**: Simplifies handling edge cases (especially when reversing from head)
4. **In-place**: No extra nodes created, only pointer manipulation
5. **Practice drawing**: Visualize pointer changes on paper to understand better
6. **Iterative vs Recursive**: Iterative uses O(1) space, recursive uses O(n) space
7. **Common pattern**: Many linked list problems use partial reversal as a sub-step

[← Previous: Fast & Slow Pointers](./04-fast-slow-pointers.md) | [Back to Index](./README.md) | [Next: Monotonic Stack →](./06-monotonic-stack.md)
