# Breadth-First Search (BFS) Pattern

## Pattern Overview

**Breadth-First Search (BFS)** is a graph/tree traversal algorithm that explores nodes level by level. It visits all nodes at depth d before visiting nodes at depth d+1, using a queue data structure.

### When to Use
- Finding shortest path in unweighted graphs
- Level-order tree traversal
- Finding minimum steps/moves
- Finding all nodes at distance k
- Web crawling, social network analysis

### Key Characteristics
- Explores level by level (breadth before depth)
- Uses queue data structure (FIFO)
- Guaranteed to find shortest path in unweighted graphs
- Space complexity: O(w) where w is maximum width

### Pattern Identification
Look for this pattern when you see:
- "Shortest path" in unweighted graph
- "Minimum number of moves/steps"
- "Level order traversal"
- "Nodes at distance k"
- "Binary tree right side view"
- "Rotting oranges"

---

## Example 1: Binary Tree Right Side View (JavaScript)

### Problem
Given the root of a binary tree, imagine yourself standing on the right side of it. Return the values of the nodes you can see ordered from top to bottom (rightmost node at each level).

**LeetCode**: [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

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
 * Get right side view of binary tree using BFS
 * @param {TreeNode} root - Root of binary tree
 * @return {number[]} - Values visible from right side
 */
function rightSideView(root) {
    if (root === null) return [];

    const result = [];
    const queue = [root];

    while (queue.length > 0) {
        const levelSize = queue.length;

        // Process all nodes at current level
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();

            // The last node at each level is visible from right
            if (i === levelSize - 1) {
                result.push(node.val);
            }

            // Add children for next level (left to right)
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
    }

    return result;
}

/**
 * Alternative: DFS approach with level tracking
 * @param {TreeNode} root
 * @return {number[]}
 */
function rightSideViewDFS(root) {
    const result = [];

    function dfs(node, level) {
        if (node === null) return;

        // First node we see at this level is the rightmost
        // (because we traverse right before left)
        if (level === result.length) {
            result.push(node.val);
        }

        // Visit right before left to get rightmost first
        dfs(node.right, level + 1);
        dfs(node.left, level + 1);
    }

    dfs(root, 0);
    return result;
}

// Helper function
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
//       1
//      / \
//     2   3
//      \   \
//       5   4
const root1 = createTree([1, 2, 3, null, 5, null, 4]);
console.log(rightSideView(root1));     // Output: [1, 3, 4]
console.log(rightSideViewDFS(root1));  // Output: [1, 3, 4]
// Explanation: From right side, you see 1, 3, 4

//     1
//    / \
//   2   3
const root2 = createTree([1, 2, 3]);
console.log(rightSideView(root2));     // Output: [1, 3]

const root3 = createTree([1]);
console.log(rightSideView(root3));     // Output: [1]
```

### Explanation

**BFS Approach**:
1. Process tree level by level using queue
2. For each level, track all nodes
3. The last node processed at each level is visible from right side
4. Add its value to result

**Visual Example**:
```
Tree:      1
         /   \
        2     3
         \     \
          5     4

Level 0: [1]
  Rightmost: 1

Level 1: [2, 3]
  Rightmost: 3

Level 2: [5, 4]
  Rightmost: 4

Right side view: [1, 3, 4]
```

---

## Example 2: Rotting Oranges (Python)

### Problem
You are given an m x n grid where each cell can have one of three values:
- 0 representing an empty cell
- 1 representing a fresh orange
- 2 representing a rotten orange

Every minute, any fresh orange that is 4-directionally adjacent to a rotten orange becomes rotten. Return the minimum number of minutes that must elapse until no cell has a fresh orange. If impossible, return -1.

**LeetCode**: [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

### Solution

```python
from typing import List
from collections import deque

class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        """
        Find minimum minutes for all oranges to rot using BFS

        Args:
            grid: m x n grid with 0 (empty), 1 (fresh), 2 (rotten)

        Returns:
            Minimum minutes, or -1 if impossible
        """
        rows, cols = len(grid), len(grid[0])
        queue = deque()
        fresh_count = 0

        # Step 1: Find all rotten oranges and count fresh ones
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == 2:
                    queue.append((r, c, 0))  # (row, col, time)
                elif grid[r][c] == 1:
                    fresh_count += 1

        # Edge case: no fresh oranges
        if fresh_count == 0:
            return 0

        # Step 2: BFS from all rotten oranges simultaneously
        directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]  # right, down, left, up
        max_time = 0

        while queue:
            r, c, time = queue.popleft()
            max_time = max(max_time, time)

            # Check all 4 adjacent cells
            for dr, dc in directions:
                nr, nc = r + dr, c + dc

                # If adjacent cell is a fresh orange
                if (0 <= nr < rows and 0 <= nc < cols and
                    grid[nr][nc] == 1):
                    # Make it rotten
                    grid[nr][nc] = 2
                    fresh_count -= 1
                    # Add to queue with incremented time
                    queue.append((nr, nc, time + 1))

        # Step 3: Check if all fresh oranges became rotten
        return max_time if fresh_count == 0 else -1

# Example usage
solution = Solution()

# Example 1
grid1 = [
    [2, 1, 1],
    [1, 1, 0],
    [0, 1, 1]
]
print(solution.orangesRotting(grid1))  # Output: 4
# Explanation:
# Minute 0: [2,1,1],[1,1,0],[0,1,1]
# Minute 1: [2,2,1],[2,1,0],[0,1,1]
# Minute 2: [2,2,2],[2,2,0],[0,1,1]
# Minute 3: [2,2,2],[2,2,0],[0,2,1]
# Minute 4: [2,2,2],[2,2,0],[0,2,2]

# Example 2
grid2 = [
    [2, 1, 1],
    [0, 1, 1],
    [1, 0, 1]
]
print(solution.orangesRotting(grid2))  # Output: -1
# Explanation: Bottom left orange can never rot

# Example 3
grid3 = [[0, 2]]
print(solution.orangesRotting(grid3))  # Output: 0
# Explanation: No fresh oranges
```

### Explanation

**Multi-source BFS**:
1. **Initialize**: Find all rotten oranges (starting points) and count fresh oranges
2. **BFS**: Process all rotten oranges level by level
   - Each level represents one minute
   - Rotten oranges infect adjacent fresh oranges
3. **Track time**: Record the time at each step
4. **Verify**: Check if all fresh oranges became rotten

**Visual Example**:
```
Initial grid:
2 1 1
1 1 0
0 1 1

Minute 0: queue = [(0,0,0)]
  Fresh count = 6

Minute 1: From (0,0), rot (0,1) and (1,0)
  2 2 1
  2 1 0
  0 1 1
  queue = [(0,1,1), (1,0,1)]
  Fresh count = 4

Minute 2: From (0,1) and (1,0), rot (0,2) and (1,1)
  2 2 2
  2 2 0
  0 1 1
  queue = [(0,2,2), (1,1,2)]
  Fresh count = 2

Minute 3: From (1,1), rot (2,1)
  2 2 2
  2 2 0
  0 2 1
  queue = [(2,1,3)]
  Fresh count = 1

Minute 4: From (2,1), rot (2,2)
  2 2 2
  2 2 0
  0 2 2
  Fresh count = 0

Result: 4 minutes
```

---

## Time & Space Complexity

### Example 1: Right Side View
- **Time Complexity**: O(n) - Visit each node once
- **Space Complexity**: O(w) - Queue size (w = max width of tree)

### Example 2: Rotting Oranges
- **Time Complexity**: O(m × n) - Visit each cell at most once
- **Space Complexity**: O(m × n) - Queue can contain all cells in worst case

---

## Common Variations

1. **Level Order Problems**
   - LeetCode: [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
   - LeetCode: [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)
   - LeetCode: [637. Average of Levels in Binary Tree](https://leetcode.com/problems/average-of-levels-in-binary-tree/)

2. **Shortest Path**
   - LeetCode: [111. Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/)
   - LeetCode: [127. Word Ladder](https://leetcode.com/problems/word-ladder/)

3. **Grid BFS**
   - LeetCode: [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)
   - LeetCode: [542. 01 Matrix](https://leetcode.com/problems/01-matrix/)

4. **Multi-source BFS**
   - LeetCode: [1162. As Far from Land as Possible](https://leetcode.com/problems/as-far-from-land-as-possible/)

---

## Practice Problems

### Easy
1. [111. Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/)
2. [993. Cousins in Binary Tree](https://leetcode.com/problems/cousins-in-binary-tree/)

### Medium
3. [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
4. [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)
5. [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)
6. [542. 01 Matrix](https://leetcode.com/problems/01-matrix/)
7. [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/)
8. [127. Word Ladder](https://leetcode.com/problems/word-ladder/)

### Hard
9. [127. Word Ladder](https://leetcode.com/problems/word-ladder/)
10. [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/)

---

## Key Takeaways

1. **Queue is key**: FIFO ensures level-by-level processing

2. **Shortest path**: BFS guarantees shortest path in unweighted graphs

3. **Level tracking**: Track nodes at each level for level-specific operations

4. **Multi-source BFS**: Start BFS from multiple sources simultaneously

5. **Grid traversal**: Use 4 or 8 directions for grid problems

6. **Space trade-off**: Uses more space than DFS for deep graphs but less for wide graphs

7. **BFS vs DFS**:
   - Use BFS for shortest path
   - Use DFS for existence of path or when memory is limited

[← Previous: Depth-First Search](./11-depth-first-search.md) | [Back to Index](./README.md) | [Next: Backtracking →](./13-backtracking.md)
