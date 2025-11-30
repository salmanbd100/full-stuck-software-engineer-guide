# Backtracking Pattern

## Pattern Overview

**Backtracking** is an algorithmic technique that explores all possible solutions by incrementally building candidates and abandoning ("backtracking") those that fail to satisfy constraints. It's essentially a refined brute force approach with pruning.

### When to Use
- Finding all permutations or combinations
- Solving constraint satisfaction problems
- Generating all valid solutions
- N-Queens, Sudoku solver
- Subset problems
- Path finding with constraints

### Key Characteristics
- Explores all possibilities systematically
- Abandons invalid paths early (pruning)
- Uses recursion with state management
- Builds solution incrementally
- Backtracks when constraint violated

### Pattern Identification
Look for this pattern when you see:
- "Find all permutations"
- "Generate all combinations"
- "Find all valid solutions"
- "N-Queens problem"
- "Sudoku solver"
- "Generate parentheses"
- "Word search"

---

## Example 1: Subsets (JavaScript)

### Problem
Given an integer array `nums` of unique elements, return all possible subsets (the power set). The solution set must not contain duplicate subsets.

**LeetCode**: [78. Subsets](https://leetcode.com/problems/subsets/)

### Solution

```javascript
/**
 * Generate all subsets using backtracking
 * @param {number[]} nums - Array of unique integers
 * @return {number[][]} - All possible subsets
 */
function subsets(nums) {
    const result = [];

    function backtrack(start, currentSubset) {
        // Add current subset to result (deep copy)
        result.push([...currentSubset]);

        // Try adding each remaining number
        for (let i = start; i < nums.length; i++) {
            // Choose: Add nums[i] to current subset
            currentSubset.push(nums[i]);

            // Explore: Recurse with next index
            backtrack(i + 1, currentSubset);

            // Unchoose: Remove nums[i] (backtrack)
            currentSubset.pop();
        }
    }

    backtrack(0, []);
    return result;
}

/**
 * Alternative: Iterative approach
 * @param {number[]} nums
 * @return {number[][]}
 */
function subsetsIterative(nums) {
    const result = [[]];  // Start with empty subset

    for (const num of nums) {
        const size = result.length;
        // For each existing subset, create new subset by adding current num
        for (let i = 0; i < size; i++) {
            result.push([...result[i], num]);
        }
    }

    return result;
}

// Example usage
console.log(subsets([1, 2, 3]));
// Output: [[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]

console.log(subsets([0]));
// Output: [[], [0]]

console.log(subsetsIterative([1, 2, 3]));
// Output: [[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]
```

### Explanation

**Backtracking Template**:
```
function backtrack(state, choices):
    if is_solution(state):
        add state to result
        return

    for choice in choices:
        if is_valid(choice):
            make_choice(choice)
            backtrack(new_state, remaining_choices)
            undo_choice(choice)  // Backtrack
```

**Decision Tree for [1,2,3]**:
```
                    []
        /           |           \
      [1]          [2]          [3]
     /   \          |
  [1,2] [1,3]     [2,3]
   /
[1,2,3]

At each node:
- Add current subset to result
- For each remaining number, explore adding it
- Backtrack by removing the number
```

**Visual Execution**:
```
Start: backtrack(0, [])
  Add [] to result

  i=0: Choose 1
    backtrack(1, [1])
      Add [1] to result

      i=1: Choose 2
        backtrack(2, [1,2])
          Add [1,2] to result

          i=2: Choose 3
            backtrack(3, [1,2,3])
              Add [1,2,3] to result
            Backtrack, remove 3
        Backtrack, remove 2

      i=2: Choose 3
        backtrack(3, [1,3])
          Add [1,3] to result
        Backtrack, remove 3
    Backtrack, remove 1

  ... continues for [2] and [3]
```

---

## Example 2: Generate Parentheses (Python)

### Problem
Given `n` pairs of parentheses, write a function to generate all combinations of well-formed parentheses.

**LeetCode**: [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)

### Solution

```python
from typing import List

class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        """
        Generate all valid parentheses combinations using backtracking

        Args:
            n: Number of pairs of parentheses

        Returns:
            List of all valid combinations
        """
        result = []

        def backtrack(current, open_count, close_count):
            """
            Build valid parentheses strings

            Args:
                current: Current string being built
                open_count: Number of '(' added so far
                close_count: Number of ')' added so far
            """
            # Base case: if we've used all parentheses
            if len(current) == 2 * n:
                result.append(current)
                return

            # Choice 1: Add '(' if we haven't used all opening brackets
            if open_count < n:
                backtrack(current + '(', open_count + 1, close_count)

            # Choice 2: Add ')' if it doesn't exceed opening brackets
            if close_count < open_count:
                backtrack(current + ')', open_count, close_count + 1)

        backtrack('', 0, 0)
        return result

# Example usage
solution = Solution()

# Example 1
print(solution.generateParenthesis(3))
# Output: ["((()))", "(()())", "(())()", "()(())", "()()()"]

# Example 2
print(solution.generateParenthesis(1))
# Output: ["()"]

# Example 3
print(solution.generateParenthesis(2))
# Output: ["(())", "()()"]
```

### Explanation

**Constraints for valid parentheses**:
1. Can only add '(' if we haven't used all n opening brackets
2. Can only add ')' if it doesn't exceed number of '(' used

**Decision Tree for n=2**:
```
                      ""
                      /
                    "("
                  /     \
              "(("       "()"
               |          |
            "(()"       "()("
               |          |
           "(())"      "()()"

At each step:
- If open_count < n: Try adding '('
- If close_count < open_count: Try adding ')'
- If length = 2n: Add to result
```

**Visual Execution for n=2**:
```
backtrack("", 0, 0)
  open_count=0 < 2, add '('

  backtrack("(", 1, 0)
    open_count=1 < 2, add '('

    backtrack("((", 2, 0)
      open_count=2 not < 2, can't add '('
      close_count=0 < 2, add ')'

      backtrack("(()", 2, 1)
        close_count=1 < 2, add ')'

        backtrack("(())", 2, 2)
          length=4, add "(())" to result

    close_count=0 < 1, add ')'

    backtrack("()", 1, 1)
      open_count=1 < 2, add '('

      backtrack("()(", 2, 1)
        close_count=1 < 2, add ')'

        backtrack("()()", 2, 2)
          length=4, add "()()" to result

Result: ["(())", "()()"]
```

---

## Time & Space Complexity

### Example 1: Subsets
- **Time Complexity**: O(n × 2^n) - 2^n subsets, each takes O(n) to copy
- **Space Complexity**: O(n) - Recursion depth

### Example 2: Generate Parentheses
- **Time Complexity**: O(4^n / √n) - Catalan number
- **Space Complexity**: O(n) - Recursion depth

---

## Common Variations

1. **Subsets and Combinations**
   - LeetCode: [78. Subsets](https://leetcode.com/problems/subsets/)
   - LeetCode: [90. Subsets II](https://leetcode.com/problems/subsets-ii/) (with duplicates)
   - LeetCode: [77. Combinations](https://leetcode.com/problems/combinations/)

2. **Permutations**
   - LeetCode: [46. Permutations](https://leetcode.com/problems/permutations/)
   - LeetCode: [47. Permutations II](https://leetcode.com/problems/permutations-ii/) (with duplicates)

3. **Constraint Satisfaction**
   - LeetCode: [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)
   - LeetCode: [51. N-Queens](https://leetcode.com/problems/n-queens/)

4. **String/Array Search**
   - LeetCode: [79. Word Search](https://leetcode.com/problems/word-search/)
   - LeetCode: [212. Word Search II](https://leetcode.com/problems/word-search-ii/)

5. **Partition Problems**
   - LeetCode: [131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

---

## Practice Problems

### Easy
1. [78. Subsets](https://leetcode.com/problems/subsets/)
2. [77. Combinations](https://leetcode.com/problems/combinations/)

### Medium
3. [46. Permutations](https://leetcode.com/problems/permutations/)
4. [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)
5. [39. Combination Sum](https://leetcode.com/problems/combination-sum/)
6. [40. Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)
7. [79. Word Search](https://leetcode.com/problems/word-search/)
8. [131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)
9. [17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

### Hard
10. [51. N-Queens](https://leetcode.com/problems/n-queens/)
11. [37. Sudoku Solver](https://leetcode.com/problems/sudoku-solver/)

---

## Key Takeaways

1. **Three-step template**:
   - **Choose**: Make a choice
   - **Explore**: Recursively explore
   - **Unchoose**: Backtrack (undo choice)

2. **State management**: Carefully track and restore state

3. **Pruning**: Abandon invalid paths early to improve efficiency

4. **Base case**: Know when to add solution to result

5. **Common patterns**:
   - Loop through choices at each step
   - Recursive call with modified state
   - Restore state after recursion

6. **Optimization**: Use constraints to prune search space

7. **Debugging tip**: Draw the decision tree to visualize the recursion

[← Previous: Breadth-First Search](./12-breadth-first-search.md) | [Back to Index](./README.md) | [Next: Dynamic Programming →](./14-dynamic-programming.md)
