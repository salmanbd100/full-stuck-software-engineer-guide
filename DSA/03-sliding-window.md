# Sliding Window Pattern

## Pattern Overview

The **Sliding Window** pattern is used to perform operations on a specific window size of an array or string. The window "slides" through the data structure by adding new elements on one side and removing elements from the other side, maintaining a contiguous sequence.

### When to Use
- Finding longest/shortest substring with specific conditions
- Finding maximum/minimum sum of subarrays of size k
- Problems involving contiguous sequences
- Optimization problems on strings or arrays

### Key Characteristics
- Maintains a window (contiguous subset) of elements
- Window size can be fixed or dynamic
- Eliminates redundant calculations by reusing previous window data
- Reduces O(n×k) or O(n²) solutions to O(n)

### Pattern Identification
Look for this pattern when you see:
- "Find the longest substring..."
- "Find the maximum sum subarray of size k"
- "Minimum window substring"
- "Find all anagrams in a string"
- Problems involving contiguous elements with constraints

---

## Example 1: Maximum Sum Subarray of Size K (JavaScript)

### Problem
Given an array of integers and a number k, find the maximum sum of any contiguous subarray of size k.

**Similar to**: [643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/)

### Solution

```javascript
/**
 * Find maximum sum of subarray of size k using sliding window
 * @param {number[]} nums - Array of integers
 * @param {number} k - Subarray size
 * @return {number} - Maximum sum of subarray of size k
 */
function maxSumSubarray(nums, k) {
    // Edge case: if array is smaller than k
    if (nums.length < k) {
        return null;
    }

    // Calculate sum of first window
    let windowSum = 0;
    for (let i = 0; i < k; i++) {
        windowSum += nums[i];
    }

    let maxSum = windowSum;

    // Slide the window through the array
    // Add new element from right, remove old element from left
    for (let i = k; i < nums.length; i++) {
        // Slide window: add new element, remove leftmost element
        windowSum = windowSum + nums[i] - nums[i - k];

        // Update maximum sum
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum;
}

// Example usage
console.log(maxSumSubarray([2, 1, 5, 1, 3, 2], 3));  // Output: 9
// Explanation: Subarray [5, 1, 3] has maximum sum 9

console.log(maxSumSubarray([2, 3, 4, 1, 5], 2));     // Output: 7
// Explanation: Subarray [3, 4] has maximum sum 7

console.log(maxSumSubarray([1, 4, 2, 10, 23, 3, 1, 0, 20], 4));  // Output: 39
// Explanation: Subarray [4, 2, 10, 23] has maximum sum 39

// Alternative solution with average
function findMaxAverage(nums, k) {
    let sum = 0;

    // Calculate first window sum
    for (let i = 0; i < k; i++) {
        sum += nums[i];
    }

    let maxSum = sum;

    // Slide window
    for (let i = k; i < nums.length; i++) {
        sum = sum + nums[i] - nums[i - k];
        maxSum = Math.max(maxSum, sum);
    }

    return maxSum / k;
}

console.log(findMaxAverage([1, 12, -5, -6, 50, 3], 4));  // Output: 12.75
```

### Explanation
1. **Fixed window size**: Window always contains exactly k elements
2. **Initial window**: Calculate sum of first k elements
3. **Sliding**:
   - Add new element entering the window (right side)
   - Subtract element leaving the window (left side)
   - This avoids recalculating the entire sum
4. **Time saved**: Instead of recalculating sum for each window (O(n×k)), we slide in O(n)

---

## Example 2: Longest Substring Without Repeating Characters (Python)

### Problem
Given a string `s`, find the length of the longest substring without repeating characters.

**LeetCode**: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

### Solution

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        """
        Find length of longest substring without repeating characters
        using dynamic sliding window

        Args:
            s: Input string

        Returns:
            Length of longest substring without repeating characters
        """
        # HashMap to store character and its most recent index
        char_index = {}

        max_length = 0
        window_start = 0

        # Expand window by moving window_end
        for window_end in range(len(s)):
            current_char = s[window_end]

            # If character is already in window, shrink window from left
            if current_char in char_index:
                # Move window_start to position after the duplicate
                # But only if it's within current window
                window_start = max(window_start, char_index[current_char] + 1)

            # Update character's index
            char_index[current_char] = window_end

            # Calculate current window size and update max
            current_length = window_end - window_start + 1
            max_length = max(max_length, current_length)

        return max_length

# Example usage
solution = Solution()

# Example 1
print(solution.lengthOfLongestSubstring("abcabcbb"))  # Output: 3
# Explanation: "abc" is the longest substring without repeating characters

# Example 2
print(solution.lengthOfLongestSubstring("bbbbb"))     # Output: 1
# Explanation: "b" is the longest substring

# Example 3
print(solution.lengthOfLongestSubstring("pwwkew"))    # Output: 3
# Explanation: "wke" is the longest substring

# Example 4
print(solution.lengthOfLongestSubstring(""))          # Output: 0
# Explanation: Empty string

# Example 5
print(solution.lengthOfLongestSubstring("abba"))      # Output: 2
# Explanation: "ab" or "ba" is the longest substring
```

### Explanation
1. **Dynamic window**: Window size changes based on constraints (no repeating characters)
2. **HashMap**: Track each character's most recent position
3. **Expand window**: Move `window_end` to include new characters
4. **Shrink window**: When duplicate found, move `window_start` past the previous occurrence
5. **Key insight**: Use `max(window_start, char_index[current_char] + 1)` to ensure we don't move `window_start` backward
6. **Update**: Always update character's index and check for new maximum length

### Visual Example
```
String: "abcabcbb"

window_start=0, window_end=0: "a" → length=1
window_start=0, window_end=1: "ab" → length=2
window_start=0, window_end=2: "abc" → length=3
window_start=1, window_end=3: "bca" → found duplicate 'a', move start
window_start=2, window_end=4: "cab" → found duplicate 'b', move start
window_start=3, window_end=5: "abc" → found duplicate 'c', move start
... and so on
```

---

## Time & Space Complexity

### Example 1: Maximum Sum Subarray
- **Time Complexity**: O(n) - Single pass through array
- **Space Complexity**: O(1) - Only storing sum variables

### Example 2: Longest Substring
- **Time Complexity**: O(n) - Each character visited at most twice (once by window_end, once by window_start)
- **Space Complexity**: O(min(n, m)) - HashMap stores at most n characters or m (charset size)

---

## Common Variations

1. **Fixed Window Size**
   - Window size remains constant
   - Used in: Max sum subarray of size k, Max average subarray
   - LeetCode: [643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/)

2. **Dynamic Window Size**
   - Window grows and shrinks based on constraints
   - Used in: Longest substring without repeating, Minimum window substring
   - LeetCode: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

3. **Substring with K Distinct Characters**
   - Find longest substring with at most k distinct characters
   - LeetCode: [340. Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)

4. **Minimum Window**
   - Find smallest window containing all required elements
   - LeetCode: [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)

5. **String Anagrams**
   - Find all anagrams in a string
   - LeetCode: [438. Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)

---

## Practice Problems

### Easy
1. [643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/)
2. [1456. Maximum Number of Vowels in a Substring of Given Length](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/)

### Medium
3. [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
4. [438. Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)
5. [424. Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)
6. [567. Permutation in String](https://leetcode.com/problems/permutation-in-string/)
7. [1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)

### Hard
8. [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)
9. [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

---

## Key Takeaways

1. **Two types**: Fixed window (constant size) and dynamic window (variable size)
2. **Efficiency**: Avoids redundant recalculation by maintaining running state
3. **HashMap helper**: Often used with hash structures to track elements in window
4. **Template approach**: Expand window → Check constraints → Shrink if needed → Update result
5. **From O(n²) to O(n)**: Eliminates nested loops for many substring/subarray problems
6. **Watch window bounds**: Carefully manage start and end pointers to avoid off-by-one errors

[← Previous: Two Pointers](./02-two-pointers.md) | [Back to Index](./README.md) | [Next: Fast & Slow Pointers →](./04-fast-slow-pointers.md)
