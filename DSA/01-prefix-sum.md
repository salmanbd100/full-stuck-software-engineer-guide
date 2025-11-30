# Prefix Sum Pattern

## Pattern Overview

The **Prefix Sum** pattern is a preprocessing technique that creates an auxiliary array where each element at index `i` contains the sum of all elements from index `0` to `i` in the original array. This allows for constant-time range sum queries.

### When to Use
- Multiple range sum queries on a static array
- Finding subarrays with a specific sum
- Problems involving cumulative sums
- When you need to calculate sum of elements between any two indices repeatedly

### Key Characteristics
- Preprocessing: O(n) time to build prefix sum array
- Query: O(1) time for range sum queries
- Space: O(n) for storing prefix sums
- The sum of elements from index `i` to `j` = `prefix[j] - prefix[i-1]`

### Pattern Identification
Look for this pattern when you see:
- "Find sum of elements in range [i, j]"
- "Count subarrays with sum equal to k"
- "Find equilibrium index"
- Multiple queries asking for subarray sums

---

## Example 1: Range Sum Query (JavaScript)

### Problem
Given an integer array `nums`, handle multiple queries of the following type:
- Calculate the sum of the elements of nums between indices `left` and `right` (inclusive) where `left <= right`.

**LeetCode**: [303. Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)

### Solution

```javascript
/**
 * Range Sum Query using Prefix Sum
 * @param {number[]} nums - Input array
 */
class NumArray {
    constructor(nums) {
        // Build prefix sum array
        // prefix[i] = sum of elements from index 0 to i-1
        this.prefix = new Array(nums.length + 1).fill(0);

        for (let i = 0; i < nums.length; i++) {
            this.prefix[i + 1] = this.prefix[i] + nums[i];
        }
    }

    /**
     * Calculate sum between left and right indices
     * @param {number} left - Start index
     * @param {number} right - End index (inclusive)
     * @return {number} Sum of elements in range
     */
    sumRange(left, right) {
        // Sum from left to right = prefix[right+1] - prefix[left]
        return this.prefix[right + 1] - this.prefix[left];
    }
}

// Example usage
const nums = [-2, 0, 3, -5, 2, -1];
const numArray = new NumArray(nums);

console.log(numArray.sumRange(0, 2)); // Output: 1 (sum of -2, 0, 3)
console.log(numArray.sumRange(2, 5)); // Output: -1 (sum of 3, -5, 2, -1)
console.log(numArray.sumRange(0, 5)); // Output: -3 (sum of all elements)
```

### Explanation
1. **Preprocessing**: Build a prefix sum array where `prefix[i]` stores the sum of elements from index 0 to i-1
2. **Query**: To get sum from index `left` to `right`, we calculate `prefix[right+1] - prefix[left]`
3. **Why it works**:
   - `prefix[right+1]` contains sum from 0 to right
   - `prefix[left]` contains sum from 0 to left-1
   - Subtracting gives us sum from left to right

---

## Example 2: Subarray Sum Equals K (Python)

### Problem
Given an array of integers `nums` and an integer `k`, return the total number of subarrays whose sum equals to `k`.

**LeetCode**: [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

### Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        """
        Count subarrays with sum equal to k using prefix sum + hashmap

        Args:
            nums: List of integers
            k: Target sum

        Returns:
            Count of subarrays with sum equal to k
        """
        # HashMap to store frequency of prefix sums
        prefix_sum_count = defaultdict(int)
        prefix_sum_count[0] = 1  # Empty subarray has sum 0

        current_sum = 0
        count = 0

        for num in nums:
            # Calculate running prefix sum
            current_sum += num

            # Check if (current_sum - k) exists in hashmap
            # If yes, it means there's a subarray ending at current index with sum k
            if (current_sum - k) in prefix_sum_count:
                count += prefix_sum_count[current_sum - k]

            # Add current prefix sum to hashmap
            prefix_sum_count[current_sum] += 1

        return count

# Example usage
solution = Solution()

# Example 1
nums1 = [1, 1, 1]
k1 = 2
print(solution.subarraySum(nums1, k1))  # Output: 2
# Subarrays: [1,1] and [1,1]

# Example 2
nums2 = [1, 2, 3]
k2 = 3
print(solution.subarraySum(nums2, k2))  # Output: 2
# Subarrays: [1,2] and [3]

# Example 3
nums3 = [1, -1, 1, -1, 1]
k3 = 0
print(solution.subarraySum(nums3, k3))  # Output: 4
# Subarrays: [1,-1], [1,-1], [-1,1], [1,-1,1,-1]
```

### Explanation
1. **Key Insight**: If `prefix_sum[i] - prefix_sum[j] = k`, then sum of subarray from `j+1` to `i` equals k
2. **Rearranging**: `prefix_sum[j] = prefix_sum[i] - k`
3. **Algorithm**:
   - Maintain a hashmap of prefix sums and their frequencies
   - For each element, check if `(current_sum - k)` exists in the map
   - If exists, add its frequency to the count (these are all valid subarrays ending at current index)
4. **Why hashmap**: Allows O(1) lookup to find how many times we've seen a particular prefix sum

---

## Time & Space Complexity

### Example 1: Range Sum Query
- **Time Complexity**:
  - Constructor: O(n) - Building prefix sum array
  - sumRange query: O(1) - Simple subtraction
- **Space Complexity**: O(n) - Storing prefix sum array

### Example 2: Subarray Sum Equals K
- **Time Complexity**: O(n) - Single pass through array
- **Space Complexity**: O(n) - HashMap storing prefix sums (worst case all unique)

---

## Common Variations

1. **2D Prefix Sum (Matrix)**
   - Extend to 2D arrays for range sum queries on matrices
   - LeetCode: [304. Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/)

2. **Product Instead of Sum**
   - Track product of elements instead of sum
   - LeetCode: [238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)

3. **Equilibrium Index**
   - Find index where left sum equals right sum
   - Use prefix and suffix sums

4. **Continuous Subarray Sum**
   - Check if subarray sum is multiple of k
   - LeetCode: [523. Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)

5. **Maximum Size Subarray Sum Equals K**
   - Find longest subarray with sum k
   - LeetCode: [325. Maximum Size Subarray Sum Equals k](https://leetcode.com/problems/maximum-size-subarray-sum-equals-k/)

---

## Practice Problems

### Easy
1. [303. Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)
2. [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/)
3. [724. Find Pivot Index](https://leetcode.com/problems/find-pivot-index/)

### Medium
4. [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)
5. [523. Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)
6. [304. Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/)
7. [930. Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/)

### Hard
8. [1074. Number of Submatrices That Sum to Target](https://leetcode.com/problems/number-of-submatrices-that-sum-to-target/)

---

## Key Takeaways

1. **Preprocessing pays off**: Spend O(n) once to enable O(1) queries
2. **Prefix sum + HashMap**: Powerful combination for subarray sum problems
3. **Formula**: `sum(i to j) = prefix[j] - prefix[i-1]`
4. **Watch for edge cases**: Empty arrays, single elements, negative numbers
5. **Alternative to nested loops**: Reduces O(n²) solutions to O(n)

[← Back to Index](./README.md) | [Next: Two Pointers →](./02-two-pointers.md)
