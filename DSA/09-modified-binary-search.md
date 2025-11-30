# Modified Binary Search Pattern

## Pattern Overview

**Modified Binary Search** adapts the classic binary search algorithm to solve problems beyond finding elements in sorted arrays. It works on sorted or partially sorted data and reduces search space by half in each iteration.

### When to Use
- Searching in rotated sorted arrays
- Finding peak elements
- Searching in 2D matrices
- Finding first/last occurrence
- Finding minimum in rotated sorted array
- Search space reduction problems

### Key Characteristics
- Works on sorted or partially sorted data
- Divides search space in half each iteration
- Time complexity: O(log n)
- Requires identifying how to eliminate half the search space

### Pattern Identification
Look for this pattern when you see:
- "Find in rotated sorted array"
- "Find peak element"
- "Search in 2D matrix"
- "Find first/last occurrence"
- "Find minimum/maximum in rotated array"
- Problems with sorted data or search space

---

## Example 1: Search in Rotated Sorted Array (JavaScript)

### Problem
Given a rotated sorted array `nums` (rotated at an unknown pivot) and a target value, return the index of the target if it exists, otherwise return -1. You must write an algorithm with O(log n) runtime complexity.

**LeetCode**: [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)

### Solution

```javascript
/**
 * Search in rotated sorted array using modified binary search
 * @param {number[]} nums - Rotated sorted array
 * @param {number} target - Target value to find
 * @return {number} - Index of target, or -1 if not found
 */
function search(nums, target) {
    let left = 0;
    let right = nums.length - 1;

    while (left <= right) {
        const mid = Math.floor((left + right) / 2);

        // Found target
        if (nums[mid] === target) {
            return mid;
        }

        // Determine which half is sorted
        // Left half is sorted
        if (nums[left] <= nums[mid]) {
            // Check if target is in sorted left half
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1;  // Search left half
            } else {
                left = mid + 1;   // Search right half
            }
        }
        // Right half is sorted
        else {
            // Check if target is in sorted right half
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1;   // Search right half
            } else {
                right = mid - 1;  // Search left half
            }
        }
    }

    return -1;  // Target not found
}

// Example usage
console.log(search([4, 5, 6, 7, 0, 1, 2], 0));     // Output: 4
// Explanation: 0 is at index 4

console.log(search([4, 5, 6, 7, 0, 1, 2], 3));     // Output: -1
// Explanation: 3 is not in array

console.log(search([1], 0));                        // Output: -1

console.log(search([1, 3], 3));                     // Output: 1

console.log(search([3, 1], 1));                     // Output: 1
```

### Explanation

**Key Insight**: Even though array is rotated, at least one half is always sorted.

**Visual Example** for `nums = [4,5,6,7,0,1,2], target = 0`:

```
Array: [4, 5, 6, 7, 0, 1, 2]
       left=0    mid=3   right=6

Iteration 1:
  mid = 3, nums[mid] = 7
  Left half [4,5,6,7] is sorted (nums[0]=4 <= nums[3]=7)
  Is target in [4,7]? No (0 is not in [4,7])
  Search right half
  left = 4, right = 6

Iteration 2:
  mid = 5, nums[mid] = 1
  Right half [1,2] is sorted (nums[5]=1 > nums[4]=0, so left not sorted)
  Is target in [1,2]? No (0 is not in [1,2])
  Search left half
  left = 4, right = 4

Iteration 3:
  mid = 4, nums[mid] = 0
  Found! Return 4
```

**Decision Tree**:
1. Compare `nums[mid]` with target → Found? Return
2. Check which half is sorted:
   - If `nums[left] <= nums[mid]` → left half sorted
   - Else → right half sorted
3. Check if target is in sorted half
4. Eliminate half that doesn't contain target

---

## Example 2: Find Minimum in Rotated Sorted Array (Python)

### Problem
Suppose an array of length `n` sorted in ascending order is rotated between 1 and `n` times. Find the minimum element.

**LeetCode**: [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

### Solution

```python
from typing import List

class Solution:
    def findMin(self, nums: List[int]) -> int:
        """
        Find minimum element in rotated sorted array

        Args:
            nums: Rotated sorted array with unique elements

        Returns:
            Minimum element in the array
        """
        left = 0
        right = len(nums) - 1

        # If array is not rotated (already sorted)
        if nums[left] < nums[right]:
            return nums[left]

        while left < right:
            mid = (left + right) // 2

            # Check if mid+1 is the minimum
            # (mid is greater than its next element)
            if mid < len(nums) - 1 and nums[mid] > nums[mid + 1]:
                return nums[mid + 1]

            # Check if mid is the minimum
            # (mid is smaller than its previous element)
            if mid > 0 and nums[mid] < nums[mid - 1]:
                return nums[mid]

            # Decide which half to search
            # If left half is sorted, minimum is in right half
            if nums[left] <= nums[mid]:
                left = mid + 1
            # Right half is sorted, minimum is in left half
            else:
                right = mid

        return nums[left]

    def findMinAlternative(self, nums: List[int]) -> int:
        """
        Alternative cleaner approach
        """
        left, right = 0, len(nums) - 1

        while left < right:
            mid = (left + right) // 2

            # If mid element is greater than right element,
            # minimum must be in right half
            if nums[mid] > nums[right]:
                left = mid + 1
            # Otherwise, minimum is in left half (including mid)
            else:
                right = mid

        return nums[left]

# Example usage
solution = Solution()

# Example 1
nums1 = [3, 4, 5, 1, 2]
print(solution.findMin(nums1))  # Output: 1
# Explanation: Original array was [1,2,3,4,5] rotated 3 times

# Example 2
nums2 = [4, 5, 6, 7, 0, 1, 2]
print(solution.findMin(nums2))  # Output: 0
# Explanation: Original array was [0,1,2,4,5,6,7] rotated 4 times

# Example 3
nums3 = [11, 13, 15, 17]
print(solution.findMin(nums3))  # Output: 11
# Explanation: Array is not rotated (or rotated 0 times)

# Example 4
nums4 = [2, 1]
print(solution.findMin(nums4))  # Output: 1

# Example 5 - Using alternative approach
nums5 = [3, 4, 5, 1, 2]
print(solution.findMinAlternative(nums5))  # Output: 1
```

### Explanation

**Key Insight**: Minimum element is the only element smaller than its previous element (inflection point).

**Visual Example** for `nums = [4,5,6,7,0,1,2]`:

```
Array: [4, 5, 6, 7, 0, 1, 2]
                      ^
                   minimum (inflection point)

Iteration 1:
  left=0, right=6, mid=3
  nums[mid]=7, nums[right]=2
  7 > 2 → minimum is in right half
  left = 4

Iteration 2:
  left=4, right=6, mid=5
  nums[mid]=1, nums[right]=2
  1 < 2 → minimum could be mid or in left half
  right = 5

Iteration 3:
  left=4, right=5, mid=4
  nums[mid]=0, nums[right]=1
  0 < 1 → minimum could be mid or in left half
  right = 4

left == right, return nums[4] = 0
```

**Comparison Logic**:
- If `nums[mid] > nums[right]`: Minimum is in right half (array is rotated at right)
- If `nums[mid] <= nums[right]`: Minimum is in left half including mid

---

## Time & Space Complexity

### Example 1: Search in Rotated Array
- **Time Complexity**: O(log n) - Binary search
- **Space Complexity**: O(1) - Constant space

### Example 2: Find Minimum
- **Time Complexity**: O(log n) - Binary search
- **Space Complexity**: O(1) - Constant space

---

## Common Variations

1. **Search in Rotated Sorted Array**
   - Find target in rotated array
   - LeetCode: [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)
   - LeetCode: [81. Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) (with duplicates)

2. **Find Minimum in Rotated Array**
   - LeetCode: [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
   - LeetCode: [154. Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) (with duplicates)

3. **First and Last Position**
   - Find first and last occurrence of element
   - LeetCode: [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

4. **Peak Element**
   - Find peak element in array
   - LeetCode: [162. Find Peak Element](https://leetcode.com/problems/find-peak-element/)

5. **Search 2D Matrix**
   - Search in row and column sorted matrix
   - LeetCode: [74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)
   - LeetCode: [240. Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/)

---

## Practice Problems

### Easy
1. [704. Binary Search](https://leetcode.com/problems/binary-search/)
2. [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/)

### Medium
3. [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)
4. [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
5. [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
6. [162. Find Peak Element](https://leetcode.com/problems/find-peak-element/)
7. [74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)
8. [240. Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/)

### Hard
9. [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

---

## Key Takeaways

1. **Classic binary search**: Foundation for all variations
2. **Identify sorted half**: Key insight for rotated array problems
3. **Invariant**: Maintain property that answer is always in search range
4. **Avoid infinite loops**: Ensure left/right pointers always make progress
5. **Edge cases**: Single element, two elements, not rotated
6. **Template**:
   ```
   while left < right:
       mid = (left + right) // 2
       if condition_to_go_right:
           left = mid + 1
       else:
           right = mid
   ```

[← Previous: Overlapping Intervals](./08-overlapping-intervals.md) | [Back to Index](./README.md) | [Next: Binary Tree Traversal →](./10-binary-tree-traversal.md)
