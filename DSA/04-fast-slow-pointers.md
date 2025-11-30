# Fast & Slow Pointers Pattern

## Pattern Overview

The **Fast & Slow Pointers** pattern (also known as the **Tortoise and Hare** algorithm) uses two pointers that move through a data structure at different speeds. This pattern is particularly useful for detecting cycles and finding middle elements in linked lists.

### When to Use
- Detecting cycles in linked lists
- Finding the middle of a linked list
- Finding the start of a cycle
- Checking if a linked list is a palindrome
- Finding the k-th element from the end

### Key Characteristics
- Two pointers move at different speeds (slow moves 1 step, fast moves 2 steps)
- If there's a cycle, fast and slow pointers will eventually meet
- Slow pointer will be at middle when fast reaches end
- Space efficient: O(1) space instead of O(n) with hash sets

### Pattern Identification
Look for this pattern when you see:
- "Detect cycle in a linked list"
- "Find the middle of a linked list"
- "Find the start of the cycle"
- "Check if linked list is palindrome"
- Problems involving linked lists without random access

---

## Example 1: Linked List Cycle Detection (JavaScript)

### Problem
Given the head of a linked list, determine if the linked list has a cycle in it. Return `true` if there is a cycle, otherwise return `false`.

**LeetCode**: [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

### Solution

```javascript
/**
 * Definition for singly-linked list node
 */
class ListNode {
    constructor(val) {
        this.val = val;
        this.next = null;
    }
}

/**
 * Detect if linked list has a cycle using fast & slow pointers
 * @param {ListNode} head - Head of the linked list
 * @return {boolean} - True if cycle exists, false otherwise
 */
function hasCycle(head) {
    // Edge case: empty list or single node
    if (!head || !head.next) {
        return false;
    }

    // Initialize slow and fast pointers
    let slow = head;
    let fast = head;

    // Move pointers until fast reaches end or they meet
    while (fast && fast.next) {
        slow = slow.next;       // Move slow pointer 1 step
        fast = fast.next.next;  // Move fast pointer 2 steps

        // If pointers meet, cycle exists
        if (slow === fast) {
            return true;
        }
    }

    // Fast pointer reached end, no cycle
    return false;
}

// Example usage
// Example 1: List with cycle [3,2,0,-4] where -4 points back to node with value 2
const node1 = new ListNode(3);
const node2 = new ListNode(2);
const node3 = new ListNode(0);
const node4 = new ListNode(-4);

node1.next = node2;
node2.next = node3;
node3.next = node4;
node4.next = node2;  // Creates cycle

console.log(hasCycle(node1));  // Output: true

// Example 2: List without cycle [1,2]
const nodeA = new ListNode(1);
const nodeB = new ListNode(2);
nodeA.next = nodeB;

console.log(hasCycle(nodeA));  // Output: false

// Example 3: Single node with cycle [1] pointing to itself
const single = new ListNode(1);
single.next = single;

console.log(hasCycle(single)); // Output: true
```

### Explanation
1. **Why it works**:
   - If there's no cycle, fast pointer reaches end (null)
   - If there's a cycle, fast pointer enters cycle first
   - Since fast moves faster, it will eventually catch up to slow pointer
   - They meet inside the cycle

2. **Analogy**: Imagine two runners on a circular track. The faster runner will eventually lap the slower runner.

3. **Time advantage**: This is O(n) time and O(1) space, compared to using a HashSet which would be O(n) space

### Finding Cycle Start Point
```javascript
/**
 * Find the node where cycle begins
 * @param {ListNode} head
 * @return {ListNode} - Node where cycle begins, or null if no cycle
 */
function detectCycle(head) {
    if (!head || !head.next) return null;

    let slow = head;
    let fast = head;

    // First, detect if cycle exists
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow === fast) {
            // Cycle detected! Now find the start
            slow = head;

            // Move both pointers one step at a time
            while (slow !== fast) {
                slow = slow.next;
                fast = fast.next;
            }

            return slow;  // This is the start of the cycle
        }
    }

    return null;  // No cycle
}
```

---

## Example 2: Middle of the Linked List (Python)

### Problem
Given the head of a singly linked list, return the middle node. If there are two middle nodes, return the second middle node.

**LeetCode**: [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

### Solution

```python
from typing import Optional

class ListNode:
    """Definition for singly-linked list node"""
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        """
        Find middle node using fast & slow pointers

        Args:
            head: Head of the linked list

        Returns:
            Middle node (second middle if even number of nodes)
        """
        # Initialize both pointers at head
        slow = head
        fast = head

        # Move fast pointer 2x speed of slow pointer
        # When fast reaches end, slow will be at middle
        while fast and fast.next:
            slow = slow.next        # Move 1 step
            fast = fast.next.next   # Move 2 steps

        # Slow pointer is now at middle
        return slow

# Example usage
def create_linked_list(values):
    """Helper function to create linked list from list of values"""
    if not values:
        return None

    head = ListNode(values[0])
    current = head

    for val in values[1:]:
        current.next = ListNode(val)
        current = current.next

    return head

def print_from_node(node):
    """Helper function to print linked list from a given node"""
    result = []
    while node:
        result.append(node.val)
        node = node.next
    return result

solution = Solution()

# Example 1: Odd number of nodes [1,2,3,4,5]
head1 = create_linked_list([1, 2, 3, 4, 5])
middle1 = solution.middleNode(head1)
print(print_from_node(middle1))  # Output: [3, 4, 5]
# Explanation: Middle node is 3

# Example 2: Even number of nodes [1,2,3,4,5,6]
head2 = create_linked_list([1, 2, 3, 4, 5, 6])
middle2 = solution.middleNode(head2)
print(print_from_node(middle2))  # Output: [4, 5, 6]
# Explanation: Two middle nodes (3 and 4), return second middle

# Example 3: Two nodes [1,2]
head3 = create_linked_list([1, 2])
middle3 = solution.middleNode(head3)
print(print_from_node(middle3))  # Output: [2]

# Example 4: Single node [1]
head4 = create_linked_list([1])
middle4 = solution.middleNode(head4)
print(print_from_node(middle4))  # Output: [1]
```

### Explanation
1. **Speed relationship**: Fast moves 2 steps for every 1 step slow moves
2. **When fast reaches end**:
   - Fast has traversed n nodes
   - Slow has traversed n/2 nodes
   - Slow is at the middle!

3. **Odd length list** (e.g., 1→2→3→4→5):
   ```
   Initial: slow=1, fast=1
   Step 1:  slow=2, fast=3
   Step 2:  slow=3, fast=5
   Step 3:  fast=null, STOP → slow=3 (middle)
   ```

4. **Even length list** (e.g., 1→2→3→4→5→6):
   ```
   Initial: slow=1, fast=1
   Step 1:  slow=2, fast=3
   Step 2:  slow=3, fast=5
   Step 3:  slow=4, fast=null, STOP → slow=4 (second middle)
   ```

---

## Time & Space Complexity

### Example 1: Cycle Detection
- **Time Complexity**: O(n) - In worst case, visit all nodes once
- **Space Complexity**: O(1) - Only two pointer variables

### Example 2: Middle Node
- **Time Complexity**: O(n) - Traverse list once (n/2 steps for slow pointer)
- **Space Complexity**: O(1) - Only two pointer variables

---

## Common Variations

1. **Cycle Detection**
   - Detect if cycle exists
   - LeetCode: [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

2. **Find Cycle Start**
   - Find where the cycle begins
   - LeetCode: [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)

3. **Middle of Linked List**
   - Find middle node
   - LeetCode: [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

4. **Palindrome Linked List**
   - Check if linked list is palindrome (find middle, reverse second half, compare)
   - LeetCode: [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

5. **Happy Number**
   - Detect cycle in number transformation
   - LeetCode: [202. Happy Number](https://leetcode.com/problems/happy-number/)

6. **Reorder List**
   - Find middle, reverse second half, merge alternately
   - LeetCode: [143. Reorder List](https://leetcode.com/problems/reorder-list/)

---

## Practice Problems

### Easy
1. [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)
2. [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)
3. [202. Happy Number](https://leetcode.com/problems/happy-number/)

### Medium
4. [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)
5. [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)
6. [143. Reorder List](https://leetcode.com/problems/reorder-list/)
7. [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

### Hard
8. [457. Circular Array Loop](https://leetcode.com/problems/circular-array-loop/)

---

## Key Takeaways

1. **Space efficient**: O(1) space compared to HashSet approach
2. **Cycle detection**: Fast will eventually meet slow if cycle exists
3. **Middle finding**: When fast reaches end, slow is at middle
4. **Speed matters**: Fast moves 2x speed of slow (key to the pattern)
5. **Watch null checks**: Always check `fast` and `fast.next` before moving fast pointer
6. **Mathematical proof**: Floyd's Cycle Detection algorithm has mathematical proof of correctness
7. **Beyond linked lists**: Can be applied to any sequence where you can detect cycles

[← Previous: Sliding Window](./03-sliding-window.md) | [Back to Index](./README.md) | [Next: In-place Reversal →](./05-linkedlist-in-place-reversal.md)
