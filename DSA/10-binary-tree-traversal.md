# Binary Tree Traversal Pattern

## Pattern Overview

**Binary Tree Traversal** involves visiting all nodes in a binary tree in a specific order. There are four main traversal types: Inorder, Preorder, Postorder (all DFS-based), and Level-order (BFS-based).

### When to Use
- Processing all nodes in a tree
- Serializing/deserializing trees
- Finding paths or validating tree properties
- Converting tree to array representation
- Tree reconstruction problems

### Key Characteristics
- **Inorder** (Left → Root → Right): Gives sorted order for BST
- **Preorder** (Root → Left → Right): Root processed first, useful for copying trees
- **Postorder** (Left → Right → Root): Children before parent, useful for deletion
- **Level-order** (BFS): Level by level, left to right

### Pattern Identification
Look for this pattern when you see:
- "Traverse a binary tree"
- "Inorder/Preorder/Postorder traversal"
- "Level order traversal"
- "Serialize/deserialize tree"
- "Convert BST to sorted array"

---

## Example 1: Binary Tree Inorder Traversal (JavaScript)

### Problem
Given the root of a binary tree, return the inorder traversal of its nodes' values (Left → Root → Right).

**LeetCode**: [94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

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
 * Inorder traversal - Recursive approach
 * @param {TreeNode} root - Root of binary tree
 * @return {number[]} - Inorder traversal values
 */
function inorderTraversal(root) {
    const result = [];

    function traverse(node) {
        if (node === null) return;

        // Left → Root → Right
        traverse(node.left);      // Visit left subtree
        result.push(node.val);    // Process current node
        traverse(node.right);     // Visit right subtree
    }

    traverse(root);
    return result;
}

/**
 * Inorder traversal - Iterative approach using stack
 * @param {TreeNode} root
 * @return {number[]}
 */
function inorderTraversalIterative(root) {
    const result = [];
    const stack = [];
    let current = root;

    while (current !== null || stack.length > 0) {
        // Go to leftmost node
        while (current !== null) {
            stack.push(current);
            current = current.left;
        }

        // Current is null, pop from stack
        current = stack.pop();
        result.push(current.val);    // Process node

        // Move to right subtree
        current = current.right;
    }

    return result;
}

// Helper function to create tree from array (level-order)
function createTree(arr) {
    if (!arr.length) return null;

    const root = new TreeNode(arr[0]);
    const queue = [root];
    let i = 1;

    while (queue.length && i < arr.length) {
        const node = queue.shift();

        if (i < arr.length && arr[i] !== null) {
            node.left = new TreeNode(arr[i]);
            queue.push(node.left);
        }
        i++;

        if (i < arr.length && arr[i] !== null) {
            node.right = new TreeNode(arr[i]);
            queue.push(node.right);
        }
        i++;
    }

    return root;
}

// Example usage
// Tree:     1
//            \
//             2
//            /
//           3
const root1 = createTree([1, null, 2, 3]);
console.log(inorderTraversal(root1));           // Output: [1, 3, 2]
console.log(inorderTraversalIterative(root1));  // Output: [1, 3, 2]

// Tree:       4
//           /   \
//          2     6
//         / \   / \
//        1   3 5   7
const root2 = createTree([4, 2, 6, 1, 3, 5, 7]);
console.log(inorderTraversal(root2));           // Output: [1, 2, 3, 4, 5, 6, 7]
// Note: For BST, inorder gives sorted order!
```

### Explanation

**Recursive Approach**:
```
For each node:
1. Recursively visit left subtree
2. Process current node (add to result)
3. Recursively visit right subtree
```

**Iterative Approach**:
```
1. Use stack to simulate recursion
2. Go as left as possible, pushing nodes to stack
3. When can't go left, pop node and process it
4. Move to right child and repeat
```

**Visual Example**:
```
Tree:     4
        /   \
       2     6
      / \   / \
     1   3 5   7

Inorder (Left → Root → Right):
1. Visit left: 1
2. Visit root: 2
3. Visit right: 3
4. Visit root: 4
5. Visit left: 5
6. Visit root: 6
7. Visit right: 7

Result: [1, 2, 3, 4, 5, 6, 7]
```

---

## Example 2: Binary Tree Level Order Traversal (Python)

### Problem
Given the root of a binary tree, return the level order traversal of its nodes' values (from left to right, level by level).

**LeetCode**: [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

### Solution

```python
from typing import Optional, List
from collections import deque

class TreeNode:
    """Definition for a binary tree node"""
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        """
        Level order traversal using BFS with queue

        Args:
            root: Root of binary tree

        Returns:
            List of lists, each containing nodes at that level
        """
        if not root:
            return []

        result = []
        queue = deque([root])

        while queue:
            level_size = len(queue)  # Number of nodes at current level
            current_level = []

            # Process all nodes at current level
            for _ in range(level_size):
                node = queue.popleft()
                current_level.append(node.val)

                # Add children to queue for next level
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)

            result.append(current_level)

        return result

    def levelOrderRecursive(self, root: Optional[TreeNode]) -> List[List[int]]:
        """
        Alternative: Recursive DFS approach with level tracking
        """
        result = []

        def dfs(node, level):
            if not node:
                return

            # Create new level list if this is first node at this level
            if level == len(result):
                result.append([])

            # Add current node to its level
            result[level].append(node.val)

            # Recursively process children at next level
            dfs(node.left, level + 1)
            dfs(node.right, level + 1)

        dfs(root, 0)
        return result

# Helper function to create tree
def create_tree(values):
    """Create binary tree from level-order list"""
    if not values:
        return None

    root = TreeNode(values[0])
    queue = deque([root])
    i = 1

    while queue and i < len(values):
        node = queue.popleft()

        if i < len(values) and values[i] is not None:
            node.left = TreeNode(values[i])
            queue.append(node.left)
        i += 1

        if i < len(values) and values[i] is not None:
            node.right = TreeNode(values[i])
            queue.append(node.right)
        i += 1

    return root

# Example usage
solution = Solution()

# Example 1
#     3
#    / \
#   9  20
#     /  \
#    15   7
root1 = create_tree([3, 9, 20, None, None, 15, 7])
print(solution.levelOrder(root1))
# Output: [[3], [9, 20], [15, 7]]

# Example 2
root2 = create_tree([1])
print(solution.levelOrder(root2))
# Output: [[1]]

# Example 3
root3 = create_tree([])
print(solution.levelOrder(root3))
# Output: []

# Example 4 - Using recursive approach
#       1
#      / \
#     2   3
#    / \
#   4   5
root4 = create_tree([1, 2, 3, 4, 5])
print(solution.levelOrderRecursive(root4))
# Output: [[1], [2, 3], [4, 5]]
```

### Explanation

**BFS Approach (Iterative)**:
```
1. Use queue to process nodes level by level
2. For each level:
   - Count nodes in queue (level_size)
   - Process exactly that many nodes
   - Add their children to queue
3. Each level's nodes are grouped together
```

**Visual Example**:
```
Tree:     3
        /   \
       9    20
           /  \
          15   7

Level 0: queue = [3]
  Process 3, add children
  Level: [3]
  queue = [9, 20]

Level 1: queue = [9, 20]
  Process 9, 20, add children
  Level: [9, 20]
  queue = [15, 7]

Level 2: queue = [15, 7]
  Process 15, 7
  Level: [15, 7]
  queue = []

Result: [[3], [9, 20], [15, 7]]
```

---

## Time & Space Complexity

### Example 1: Inorder Traversal
- **Time Complexity**: O(n) - Visit each node once
- **Space Complexity**:
  - Recursive: O(h) - Call stack depth (h = height)
  - Iterative: O(h) - Stack size

### Example 2: Level Order Traversal
- **Time Complexity**: O(n) - Visit each node once
- **Space Complexity**: O(w) - Queue size (w = max width of tree)

---

## Common Variations

1. **Inorder Traversal** (Left → Root → Right)
   - LeetCode: [94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

2. **Preorder Traversal** (Root → Left → Right)
   - LeetCode: [144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

3. **Postorder Traversal** (Left → Right → Root)
   - LeetCode: [145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

4. **Level Order Traversal** (BFS)
   - LeetCode: [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
   - LeetCode: [107. Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/) (bottom-up)

5. **Zigzag Level Order**
   - LeetCode: [103. Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)

6. **Vertical Order Traversal**
   - LeetCode: [314. Binary Tree Vertical Order Traversal](https://leetcode.com/problems/binary-tree-vertical-order-traversal/)

---

## Practice Problems

### Easy
1. [94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)
2. [144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)
3. [145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)
4. [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

### Medium
5. [103. Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)
6. [107. Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)
7. [314. Binary Tree Vertical Order Traversal](https://leetcode.com/problems/binary-tree-vertical-order-traversal/)
8. [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

---

## Key Takeaways

1. **Four main types**:
   - **Inorder**: Left → Root → Right (BST → sorted)
   - **Preorder**: Root → Left → Right (copying tree)
   - **Postorder**: Left → Right → Root (deleting tree)
   - **Level-order**: Level by level (BFS)

2. **Implementation choices**:
   - DFS traversals: Recursive (simple) or Iterative (stack)
   - Level-order: Queue-based BFS

3. **BST property**: Inorder traversal of BST gives sorted array

4. **Space complexity**: O(h) for DFS, O(w) for BFS

5. **Common interview trick**: "Can you do it iteratively without recursion?"

[← Previous: Modified Binary Search](./09-modified-binary-search.md) | [Back to Index](./README.md) | [Next: Depth-First Search →](./11-depth-first-search.md)
