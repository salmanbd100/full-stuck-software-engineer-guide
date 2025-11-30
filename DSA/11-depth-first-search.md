# Depth-First Search (DFS) Pattern

## Pattern Overview

**Depth-First Search (DFS)** is a graph/tree traversal algorithm that explores as far as possible along each branch before backtracking. It goes deep into a tree/graph before exploring siblings.

### When to Use
- Tree path problems
- Graph connectivity and cycle detection
- Topological sorting
- Finding all paths
- Maze solving
- Backtracking problems

### Key Characteristics
- Explores depth before breadth
- Uses stack (explicit or implicit via recursion)
- Space complexity: O(h) where h is height/depth
- Can be implemented recursively or iteratively

### Pattern Identification
Look for this pattern when you see:
- "Find all paths"
- "Validate binary search tree"
- "Path sum"
- "Number of islands"
- "Course schedule" (topological sort)
- "Surrounded regions"

---

## Example 1: Path Sum (JavaScript)

### Problem
Given the root of a binary tree and an integer `targetSum`, return `true` if the tree has a root-to-leaf path such that adding up all the values along the path equals `targetSum`.

**LeetCode**: [112. Path Sum](https://leetcode.com/problems/path-sum/)

### Solution

```javascript
/**
 * Definition for a binary tree node
 */
class TreeNode {
    constructor(val, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

/**
 * Check if tree has path with given sum using DFS
 * @param {TreeNode} root - Root of binary tree
 * @param {number} targetSum - Target sum to find
 * @return {boolean} - True if path exists
 */
function hasPathSum(root, targetSum) {
    // Base case: empty tree
    if (root === null) {
        return false;
    }

    // Base case: leaf node
    // Check if current path sum equals target
    if (root.left === null && root.right === null) {
        return root.val === targetSum;
    }

    // Recursive case: explore left and right subtrees
    // Subtract current value from target for children
    const remainingSum = targetSum - root.val;

    return hasPathSum(root.left, remainingSum) ||
           hasPathSum(root.right, remainingSum);
}

/**
 * Iterative DFS approach using stack
 * @param {TreeNode} root
 * @param {number} targetSum
 * @return {boolean}
 */
function hasPathSumIterative(root, targetSum) {
    if (root === null) return false;

    // Stack stores [node, currentSum] pairs
    const stack = [[root, root.val]];

    while (stack.length > 0) {
        const [node, currentSum] = stack.pop();

        // Check if it's a leaf node with target sum
        if (node.left === null && node.right === null && currentSum === targetSum) {
            return true;
        }

        // Add right child to stack
        if (node.right !== null) {
            stack.push([node.right, currentSum + node.right.val]);
        }

        // Add left child to stack
        if (node.left !== null) {
            stack.push([node.left, currentSum + node.left.val]);
        }
    }

    return false;
}

// Helper function to create tree
function createTree(values) {
    if (!values || values.length === 0) return null;

    const root = new TreeNode(values[0]);
    const queue = [root];
    let i = 1;

    while (queue.length && i < values.length) {
        const node = queue.shift();

        if (i < values.length && values[i] !== null) {
            node.left = new TreeNode(values[i]);
            queue.push(node.left);
        }
        i++;

        if (i < values.length && values[i] !== null) {
            node.right = new TreeNode(values[i]);
            queue.push(node.right);
        }
        i++;
    }

    return root;
}

// Example usage
//       5
//      / \
//     4   8
//    /   / \
//   11  13  4
//  /  \      \
// 7    2      1
const root1 = createTree([5, 4, 8, 11, null, 13, 4, 7, 2, null, null, null, 1]);
console.log(hasPathSum(root1, 22));  // Output: true
// Path: 5 → 4 → 11 → 2 = 22

console.log(hasPathSum(root1, 26));  // Output: true
// Path: 5 → 4 → 11 → 7 = 27 (false), or 5 → 8 → 13 = 26 (true)

const root2 = createTree([1, 2, 3]);
console.log(hasPathSum(root2, 5));   // Output: false

const root3 = createTree([]);
console.log(hasPathSum(root3, 0));   // Output: false
```

### Explanation

**DFS Approach**:
1. Start at root with full target sum
2. At each node, subtract its value from remaining sum
3. When reaching a leaf, check if remaining sum equals leaf value
4. Explore left and right subtrees (depth-first)

**Visual Example**:
```
Tree:      5
         /   \
        4     8
       /     / \
      11    13  4
     /  \        \
    7    2        1

Target: 22

DFS Path 1: 5 → 4 → 11 → 7
  5: remaining = 22 - 5 = 17
  4: remaining = 17 - 4 = 13
  11: remaining = 13 - 11 = 2
  7: remaining = 2 - 7 = -5 (leaf, not equal) ✗

DFS Path 2: 5 → 4 → 11 → 2
  5: remaining = 22 - 5 = 17
  4: remaining = 17 - 4 = 13
  11: remaining = 13 - 11 = 2
  2: remaining = 2 - 2 = 0 (leaf, equals 0) ✓

Found path! Return true
```

---

## Example 2: Number of Islands (Python)

### Problem
Given an `m x n` 2D binary grid which represents a map of '1's (land) and '0's (water), return the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically.

**LeetCode**: [200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

### Solution

```python
from typing import List

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        """
        Count number of islands using DFS

        Args:
            grid: 2D grid of '1' (land) and '0' (water)

        Returns:
            Number of islands
        """
        if not grid or not grid[0]:
            return 0

        rows, cols = len(grid), len(grid[0])
        islands = 0

        def dfs(r, c):
            """Mark all connected land cells as visited"""
            # Base cases: out of bounds or water or already visited
            if (r < 0 or r >= rows or
                c < 0 or c >= cols or
                grid[r][c] == '0'):
                return

            # Mark current cell as visited (change to '0')
            grid[r][c] = '0'

            # Explore all 4 directions (up, down, left, right)
            dfs(r - 1, c)  # up
            dfs(r + 1, c)  # down
            dfs(r, c - 1)  # left
            dfs(r, c + 1)  # right

        # Iterate through each cell in the grid
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == '1':
                    islands += 1      # Found new island
                    dfs(r, c)         # Mark entire island as visited

        return islands

    def numIslandsIterative(self, grid: List[List[str]]) -> int:
        """
        Alternative: Iterative DFS using stack
        """
        if not grid or not grid[0]:
            return 0

        rows, cols = len(grid), len(grid[0])
        islands = 0

        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == '1':
                    islands += 1

                    # Use stack for iterative DFS
                    stack = [(r, c)]

                    while stack:
                        curr_r, curr_c = stack.pop()

                        # Skip if out of bounds or water
                        if (curr_r < 0 or curr_r >= rows or
                            curr_c < 0 or curr_c >= cols or
                            grid[curr_r][curr_c] == '0'):
                            continue

                        # Mark as visited
                        grid[curr_r][curr_c] = '0'

                        # Add all 4 neighbors to stack
                        stack.append((curr_r - 1, curr_c))  # up
                        stack.append((curr_r + 1, curr_c))  # down
                        stack.append((curr_r, curr_c - 1))  # left
                        stack.append((curr_r, curr_c + 1))  # right

        return islands

# Example usage
solution = Solution()

# Example 1
grid1 = [
    ["1","1","1","1","0"],
    ["1","1","0","1","0"],
    ["1","1","0","0","0"],
    ["0","0","0","0","0"]
]
print(solution.numIslands(grid1))  # Output: 1
# One connected island

# Example 2
grid2 = [
    ["1","1","0","0","0"],
    ["1","1","0","0","0"],
    ["0","0","1","0","0"],
    ["0","0","0","1","1"]
]
print(solution.numIslands(grid2))  # Output: 3
# Three separate islands

# Example 3 - Using iterative approach
grid3 = [
    ["1","0","1"],
    ["0","1","0"],
    ["1","0","1"]
]
print(solution.numIslandsIterative(grid3))  # Output: 5
# Five separate islands (each '1' is isolated)
```

### Explanation

**Algorithm**:
1. Iterate through each cell in grid
2. When find '1' (unvisited land):
   - Increment island count
   - Use DFS to mark all connected land cells as visited
3. DFS explores all 4 directions from current cell
4. Mark visited cells as '0' to avoid recounting

**Visual Example**:
```
Grid:
1 1 1 1 0
1 1 0 1 0
1 1 0 0 0
0 0 0 0 0

Step 1: Find '1' at (0,0)
  Island count = 1
  DFS marks all connected 1s:

  0 0 0 0 0
  0 0 0 0 0
  0 0 0 0 0
  0 0 0 0 0

DFS Traversal from (0,0):
  (0,0) → (0,1) → (0,2) → (0,3) → (1,3) →
  (1,0) → (1,1) → (2,0) → (2,1)

All cells visited, final count: 1 island
```

---

## Time & Space Complexity

### Example 1: Path Sum
- **Time Complexity**: O(n) - Visit each node once in worst case
- **Space Complexity**: O(h) - Recursion depth (h = tree height)

### Example 2: Number of Islands
- **Time Complexity**: O(m × n) - Visit each cell once
- **Space Complexity**: O(m × n) - Worst case recursion depth (entire grid is one island)

---

## Common Variations

1. **Path Sum Problems**
   - LeetCode: [112. Path Sum](https://leetcode.com/problems/path-sum/)
   - LeetCode: [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/) (find all paths)
   - LeetCode: [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/) (any path)

2. **Tree Validation**
   - LeetCode: [98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)

3. **Graph Problems**
   - LeetCode: [200. Number of Islands](https://leetcode.com/problems/number-of-islands/)
   - LeetCode: [695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/)
   - LeetCode: [130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

4. **Clone Graph**
   - LeetCode: [133. Clone Graph](https://leetcode.com/problems/clone-graph/)

---

## Practice Problems

### Easy
1. [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
2. [112. Path Sum](https://leetcode.com/problems/path-sum/)
3. [110. Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)

### Medium
4. [200. Number of Islands](https://leetcode.com/problems/number-of-islands/)
5. [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/)
6. [98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
7. [230. Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)
8. [695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/)
9. [417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)

### Hard
10. [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

---

## Key Takeaways

1. **Go deep first**: Explore as far as possible before backtracking

2. **Implementation**:
   - Recursive: Clean and simple
   - Iterative: Use explicit stack

3. **Common use cases**:
   - Finding paths in trees
   - Graph connectivity
   - Cycle detection
   - Topological sorting

4. **Marking visited**: Important in graphs to avoid infinite loops

5. **Directions**: For grids, explore 4 or 8 directions

6. **Time vs Space**: DFS uses less memory than BFS for wide graphs

[← Previous: Binary Tree Traversal](./10-binary-tree-traversal.md) | [Back to Index](./README.md) | [Next: Breadth-First Search →](./12-breadth-first-search.md)
