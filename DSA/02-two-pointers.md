# Two Pointers Pattern

## Pattern Overview

The **Two Pointers** pattern uses two pointers to iterate through a data structure (usually an array or linked list) in a coordinated way. The pointers can move towards each other, in the same direction, or at different speeds.

### When to Use
- Working with sorted arrays or linked lists
- Finding pairs or triplets with specific properties
- Removing duplicates in-place
- Palindrome problems
- Partitioning arrays

### Key Characteristics
- Two pointers start at different positions (start/end, or both at start)
- Pointers move based on certain conditions
- Often eliminates the need for nested loops
- Reduces time complexity from O(n²) to O(n)

### Pattern Identification
Look for this pattern when you see:
- "Find a pair that sums to target"
- "Remove duplicates from sorted array"
- "Check if string is a palindrome"
- "Sort array with 0s, 1s, and 2s"
- Problems involving sorted arrays

---

## Example 1: Two Sum II - Sorted Array (JavaScript)

### Problem
Given a **1-indexed** array of integers `numbers` that is already sorted in non-decreasing order, find two numbers such that they add up to a specific `target` number. Return the indices of the two numbers (1-indexed).

**LeetCode**: [167. Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

### Solution

```javascript
/**
 * Find two numbers that add up to target using two pointers
 * @param {number[]} numbers - Sorted array of integers
 * @param {number} target - Target sum
 * @return {number[]} - 1-indexed positions of the two numbers
 */
function twoSum(numbers, target) {
    // Initialize two pointers
    let left = 0;                    // Start pointer at beginning
    let right = numbers.length - 1;  // End pointer at end

    while (left < right) {
        const currentSum = numbers[left] + numbers[right];

        if (currentSum === target) {
            // Found the pair! Return 1-indexed positions
            return [left + 1, right + 1];
        } else if (currentSum < target) {
            // Sum is too small, move left pointer right to increase sum
            left++;
        } else {
            // Sum is too large, move right pointer left to decrease sum
            right--;
        }
    }

    // No solution found (problem guarantees a solution exists)
    return [];
}

// Example usage
console.log(twoSum([2, 7, 11, 15], 9));     // Output: [1, 2]
// Explanation: numbers[0] + numbers[1] = 2 + 7 = 9

console.log(twoSum([2, 3, 4], 6));          // Output: [1, 3]
// Explanation: numbers[0] + numbers[2] = 2 + 4 = 6

console.log(twoSum([-1, 0], -1));           // Output: [1, 2]
// Explanation: numbers[0] + numbers[1] = -1 + 0 = -1
```

### Explanation
1. **Setup**: Place one pointer at the start (`left`) and one at the end (`right`)
2. **Calculate sum**: Add values at both pointers
3. **Decision**:
   - If sum equals target → Found! Return indices
   - If sum < target → Need larger sum, move `left` pointer right
   - If sum > target → Need smaller sum, move `right` pointer left
4. **Why it works**: Array is sorted, so moving pointers strategically adjusts the sum
5. **Time saved**: No need to check all pairs (O(n²)), just one pass (O(n))

---

## Example 2: Container With Most Water (Python)

### Problem
Given `n` non-negative integers `height` where each represents a point at coordinate `(i, height[i])`, find two lines that together with the x-axis form a container that holds the most water.

**LeetCode**: [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

### Solution

```python
from typing import List

class Solution:
    def maxArea(self, height: List[int]) -> int:
        """
        Find maximum water area using two pointers

        Args:
            height: List of non-negative integers representing heights

        Returns:
            Maximum area of water that can be contained
        """
        # Initialize pointers at both ends
        left = 0
        right = len(height) - 1
        max_area = 0

        while left < right:
            # Calculate width between pointers
            width = right - left

            # Area is limited by the shorter line
            # Area = width × min(height[left], height[right])
            current_area = width * min(height[left], height[right])

            # Update maximum area
            max_area = max(max_area, current_area)

            # Move the pointer pointing to shorter line
            # This is the key insight: moving the shorter line might find a taller one
            # Moving the taller line will only decrease area (width decreases, height can't increase)
            if height[left] < height[right]:
                left += 1
            else:
                right -= 1

        return max_area

# Example usage
solution = Solution()

# Example 1
height1 = [1, 8, 6, 2, 5, 4, 8, 3, 7]
print(solution.maxArea(height1))  # Output: 49
# Explanation: Lines at index 1 (height=8) and index 8 (height=7)
# Area = 7 × min(8, 7) = 49

# Example 2
height2 = [1, 1]
print(solution.maxArea(height2))  # Output: 1
# Explanation: Only two lines, area = 1 × min(1, 1) = 1

# Example 3
height3 = [4, 3, 2, 1, 4]
print(solution.maxArea(height3))  # Output: 16
# Explanation: Lines at index 0 and 4 (both height=4)
# Area = 4 × min(4, 4) = 16
```

### Explanation
1. **Setup**: Start with widest possible container (left=0, right=n-1)
2. **Area calculation**: `width × min(left_height, right_height)`
3. **Key insight**: Area is limited by the shorter line
4. **Greedy approach**:
   - Always move the pointer pointing to the shorter line
   - Why? Moving the taller line can't increase area (width decreases, and min height can't increase)
   - Moving the shorter line might find a taller line, potentially increasing area
5. **Termination**: When pointers meet, we've checked all possibilities

---

## Time & Space Complexity

### Example 1: Two Sum II
- **Time Complexity**: O(n) - Single pass with two pointers
- **Space Complexity**: O(1) - Only using two pointer variables

### Example 2: Container With Most Water
- **Time Complexity**: O(n) - Single pass through array
- **Space Complexity**: O(1) - Constant extra space

---

## Common Variations

1. **Opposite Direction (Converging Pointers)**
   - Start from both ends, move towards each other
   - Used in: Two Sum II, Container With Most Water, Valid Palindrome
   - LeetCode: [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

2. **Same Direction (Fast & Slow - different from Fast & Slow Pointers pattern)**
   - Both pointers start at the beginning, move at different rates
   - Used in: Remove duplicates, Move zeros
   - LeetCode: [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

3. **Three Pointers**
   - Extension to three pointers for triplet problems
   - LeetCode: [15. 3Sum](https://leetcode.com/problems/3sum/)

4. **Partition Pattern**
   - Partition array based on condition (Dutch National Flag)
   - LeetCode: [75. Sort Colors](https://leetcode.com/problems/sort-colors/)

5. **Sliding Window Variant**
   - Dynamic window size using two pointers
   - LeetCode: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

---

## Practice Problems

### Easy
1. [167. Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)
2. [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
3. [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
4. [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)

### Medium
5. [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
6. [15. 3Sum](https://leetcode.com/problems/3sum/)
7. [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/)
8. [75. Sort Colors](https://leetcode.com/problems/sort-colors/)
9. [80. Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)

### Hard
10. [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

---

## Key Takeaways

1. **When array is sorted**: Two pointers is often the optimal approach
2. **Opposite direction**: Start from both ends when looking for pairs/sums
3. **Same direction**: Use when modifying array in-place or removing elements
4. **Greedy decisions**: Move pointers based on which movement could improve the answer
5. **Space efficiency**: Usually O(1) space, making it very efficient
6. **From O(n²) to O(n)**: Eliminates need for nested loops in many scenarios

[← Previous: Prefix Sum](./01-prefix-sum.md) | [Back to Index](./README.md) | [Next: Sliding Window →](./03-sliding-window.md)
