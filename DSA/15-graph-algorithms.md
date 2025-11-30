# Graph Algorithms Pattern

## Pattern Overview

**Graph Algorithms** involve traversing and analyzing graph structures (vertices/nodes connected by edges). Graphs can be directed/undirected, weighted/unweighted, cyclic/acyclic, and represent many real-world problems.

### When to Use
- Network/social network problems
- Path finding (shortest path, all paths)
- Cycle detection
- Topological sorting
- Connected components
- Dependency resolution

### Key Characteristics
- Graph representation: Adjacency list, adjacency matrix, edge list
- Common algorithms: DFS, BFS, Dijkstra, Union-Find
- Can be directed or undirected
- May have weights on edges

### Pattern Identification
Look for this pattern when you see:
- "Find shortest path"
- "Course prerequisites" (topological sort)
- "Network delay time" (weighted shortest path)
- "Detect cycle"
- "Connected components"
- "Clone graph"

---

## Example 1: Course Schedule (JavaScript)

### Problem
There are `numCourses` courses labeled from 0 to numCourses-1. You are given an array `prerequisites` where `prerequisites[i] = [a, b]` indicates you must take course b before course a. Return `true` if you can finish all courses (no cycle in dependency graph).

**LeetCode**: [207. Course Schedule](https://leetcode.com/problems/course-schedule/)

### Solution

```javascript
/**
 * Detect cycle in directed graph using DFS (topological sort approach)
 * @param {number} numCourses - Number of courses
 * @param {number[][]} prerequisites - [course, prerequisite] pairs
 * @return {boolean} - True if can finish all courses
 */
function canFinish(numCourses, prerequisites) {
    // Build adjacency list
    const graph = Array.from({ length: numCourses }, () => []);

    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
    }

    // Track visited states: 0=unvisited, 1=visiting, 2=visited
    const visited = new Array(numCourses).fill(0);

    function hasCycle(course) {
        if (visited[course] === 1) {
            // Currently visiting - found cycle!
            return true;
        }

        if (visited[course] === 2) {
            // Already fully processed
            return false;
        }

        // Mark as visiting
        visited[course] = 1;

        // Check all neighbors
        for (const neighbor of graph[course]) {
            if (hasCycle(neighbor)) {
                return true;
            }
        }

        // Mark as visited (fully processed)
        visited[course] = 2;
        return false;
    }

    // Check each course for cycles
    for (let i = 0; i < numCourses; i++) {
        if (hasCycle(i)) {
            return false;  // Cycle detected
        }
    }

    return true;  // No cycles, can finish all courses
}

/**
 * Alternative: BFS approach (Kahn's algorithm for topological sort)
 * @param {number} numCourses
 * @param {number[][]} prerequisites
 * @return {boolean}
 */
function canFinishBFS(numCourses, prerequisites) {
    // Build graph and indegree count
    const graph = Array.from({ length: numCourses }, () => []);
    const indegree = new Array(numCourses).fill(0);

    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
        indegree[course]++;
    }

    // Start with courses that have no prerequisites
    const queue = [];
    for (let i = 0; i < numCourses; i++) {
        if (indegree[i] === 0) {
            queue.push(i);
        }
    }

    let processed = 0;

    while (queue.length > 0) {
        const course = queue.shift();
        processed++;

        // Remove this course from graph
        for (const neighbor of graph[course]) {
            indegree[neighbor]--;

            // If neighbor has no more prerequisites, add to queue
            if (indegree[neighbor] === 0) {
                queue.push(neighbor);
            }
        }
    }

    // If processed all courses, no cycle exists
    return processed === numCourses;
}

// Example usage
console.log(canFinish(2, [[1, 0]]));  // Output: true
// Explanation: Take course 0, then course 1

console.log(canFinish(2, [[1, 0], [0, 1]]));  // Output: false
// Explanation: Circular dependency

console.log(canFinishBFS(4, [[1, 0], [2, 0], [3, 1], [3, 2]]));  // Output: true
// Explanation: Valid order exists: 0 → 1 → 2 → 3 or 0 → 2 → 1 → 3
```

### Explanation

**DFS Cycle Detection**:
```
Three states for each node:
0 (white) = unvisited
1 (gray) = currently visiting (in recursion stack)
2 (black) = fully visited

If we encounter a gray node, we found a cycle!
```

**Visual Example**:
```
Graph: 0 → 1 → 2
       ↓
       3

Prerequisites: [[1,0], [2,1], [3,0]]

DFS from 0:
  Visit 0 (mark gray)
    Visit 1 (mark gray)
      Visit 2 (mark gray)
      2 has no neighbors (mark black)
    1 done (mark black)
    Visit 3 (mark gray)
    3 has no neighbors (mark black)
  0 done (mark black)

No gray nodes revisited → No cycle → Can finish
```

**Kahn's Algorithm (BFS)**:
1. Calculate indegree (incoming edges) for each node
2. Add nodes with indegree 0 to queue
3. Process queue:
   - Remove node and decrease indegree of neighbors
   - Add neighbors with indegree 0 to queue
4. If processed all nodes → no cycle

---

## Example 2: Network Delay Time (Python)

### Problem
You are given a network of `n` nodes labeled 1 to n, and `times`, an array of travel times as directed edges `times[i] = (u, v, w)` where `u` is source, `v` is target, and `w` is the time for signal to travel. Send signal from node `k`. Return minimum time for all nodes to receive signal, or -1 if impossible.

**LeetCode**: [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/)

### Solution

```python
from typing import List
import heapq
from collections import defaultdict

class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        """
        Find shortest path to all nodes using Dijkstra's algorithm

        Args:
            times: Directed weighted edges [source, target, time]
            n: Number of nodes (1 to n)
            k: Starting node

        Returns:
            Minimum time for signal to reach all nodes, or -1 if impossible
        """
        # Build adjacency list
        graph = defaultdict(list)
        for u, v, w in times:
            graph[u].append((v, w))  # (neighbor, weight)

        # Dijkstra's algorithm
        min_heap = [(0, k)]  # (time, node)
        visited = set()
        max_time = 0

        while min_heap:
            time, node = heapq.heappop(min_heap)

            # Skip if already visited
            if node in visited:
                continue

            # Mark as visited and update max time
            visited.add(node)
            max_time = max(max_time, time)

            # Explore neighbors
            for neighbor, weight in graph[node]:
                if neighbor not in visited:
                    heapq.heappush(min_heap, (time + weight, neighbor))

        # Check if all nodes were visited
        return max_time if len(visited) == n else -1

    def networkDelayTimeBellmanFord(self, times: List[List[int]], n: int, k: int) -> int:
        """
        Alternative: Bellman-Ford algorithm
        Works with negative weights (though not needed here)
        """
        # Initialize distances
        dist = [float('inf')] * (n + 1)
        dist[k] = 0

        # Relax edges n-1 times
        for _ in range(n - 1):
            for u, v, w in times:
                if dist[u] != float('inf') and dist[u] + w < dist[v]:
                    dist[v] = dist[u] + w

        # Find maximum distance
        max_dist = max(dist[1:])
        return max_dist if max_dist != float('inf') else -1

# Example usage
solution = Solution()

# Example 1
times1 = [[2, 1, 1], [2, 3, 1], [3, 4, 1]]
n1 = 4
k1 = 2
print(solution.networkDelayTime(times1, n1, k1))  # Output: 2
# Explanation:
# Node 2 → Node 1: time 1
# Node 2 → Node 3: time 1
# Node 3 → Node 4: time 1 (total from 2: 2)
# Max time: 2

# Example 2
times2 = [[1, 2, 1]]
n2 = 2
k2 = 1
print(solution.networkDelayTime(times2, n2, k2))  # Output: 1

# Example 3
times3 = [[1, 2, 1]]
n3 = 2
k3 = 2
print(solution.networkDelayTime(times3, n3, k3))  # Output: -1
# Explanation: Node 1 is unreachable from node 2

# Example 4 - Using Bellman-Ford
times4 = [[2, 1, 1], [2, 3, 1], [3, 4, 1]]
n4 = 4
k4 = 2
print(solution.networkDelayTimeBellmanFord(times4, n4, k4))  # Output: 2
```

### Explanation

**Dijkstra's Algorithm**:
1. Use min-heap to always process nearest unvisited node
2. Track visited nodes to avoid reprocessing
3. For each node, explore neighbors and update distances
4. Return max time when all nodes visited

**Visual Example**:
```
Graph (k=2):
  2 --1--> 1
  |
  1
  ↓
  3 --1--> 4

Step 1: Start at node 2 (time=0)
  heap = [(0, 2)]
  visited = {}

Step 2: Process node 2
  Visit neighbors: 1 (time=1), 3 (time=1)
  heap = [(1, 1), (1, 3)]
  visited = {2}
  max_time = 0

Step 3: Process node 1 (time=1)
  No unvisited neighbors
  heap = [(1, 3)]
  visited = {2, 1}
  max_time = 1

Step 4: Process node 3 (time=1)
  Visit neighbor: 4 (time=2)
  heap = [(2, 4)]
  visited = {2, 1, 3}
  max_time = 1

Step 5: Process node 4 (time=2)
  No unvisited neighbors
  heap = []
  visited = {2, 1, 3, 4}
  max_time = 2

Result: 2 (all nodes visited, max time is 2)
```

---

## Time & Space Complexity

### Example 1: Course Schedule
- **DFS approach**:
  - Time: O(V + E) - Visit each vertex and edge once
  - Space: O(V + E) - Graph + recursion stack
- **BFS (Kahn's) approach**:
  - Time: O(V + E)
  - Space: O(V + E) - Graph + queue

### Example 2: Network Delay Time
- **Dijkstra's**:
  - Time: O((V + E) log V) - Heap operations
  - Space: O(V + E) - Graph + heap
- **Bellman-Ford**:
  - Time: O(V × E) - Relax all edges V times
  - Space: O(V) - Distance array

---

## Common Variations

1. **Topological Sort**
   - LeetCode: [207. Course Schedule](https://leetcode.com/problems/course-schedule/)
   - LeetCode: [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

2. **Shortest Path**
   - LeetCode: [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/)
   - LeetCode: [787. Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

3. **Union-Find (Disjoint Set)**
   - LeetCode: [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) (alternative solution)
   - LeetCode: [684. Redundant Connection](https://leetcode.com/problems/redundant-connection/)
   - LeetCode: [547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/)

4. **Clone/Copy Graph**
   - LeetCode: [133. Clone Graph](https://leetcode.com/problems/clone-graph/)

---

## Practice Problems

### Medium
1. [207. Course Schedule](https://leetcode.com/problems/course-schedule/)
2. [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)
3. [133. Clone Graph](https://leetcode.com/problems/clone-graph/)
4. [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/)
5. [323. Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/)
6. [684. Redundant Connection](https://leetcode.com/problems/redundant-connection/)
7. [261. Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/)

### Hard
8. [787. Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)
9. [269. Alien Dictionary](https://leetcode.com/problems/alien-dictionary/)

---

## Key Takeaways

1. **Graph representation**:
   - **Adjacency list**: Most common, space-efficient for sparse graphs
   - **Adjacency matrix**: Fast edge lookup, O(V²) space

2. **Common algorithms**:
   - **DFS**: Cycle detection, topological sort, path finding
   - **BFS**: Shortest path (unweighted), level-order
   - **Dijkstra**: Shortest path (weighted, non-negative)
   - **Bellman-Ford**: Shortest path (works with negative weights)
   - **Union-Find**: Connected components, cycle detection

3. **Cycle detection**:
   - Directed graph: Use three-state DFS (white/gray/black)
   - Undirected graph: Use parent tracking in DFS

4. **Topological sort**: Only exists in DAG (Directed Acyclic Graph)

5. **Visited tracking**: Essential to avoid infinite loops

6. **Weighted vs Unweighted**:
   - Unweighted: BFS finds shortest path
   - Weighted: Use Dijkstra or Bellman-Ford

[← Previous: Dynamic Programming](./14-dynamic-programming.md) | [Back to Index](./README.md)
