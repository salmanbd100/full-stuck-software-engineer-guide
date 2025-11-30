# Top 'K' Elements Pattern

## Pattern Overview

The **Top 'K' Elements** pattern finds the k largest, smallest, or most frequent elements in a dataset. This pattern typically uses heaps (priority queues) to efficiently maintain and retrieve the top k elements.

### When to Use
- Finding k largest or smallest elements
- Finding k most/least frequent elements
- Finding k closest points to origin
- Streaming data where you need to maintain top k
- Sorting only the first k elements

### Key Characteristics
- Uses Min-Heap for k largest elements (or Max-Heap for k smallest)
- Heap size is maintained at k
- More efficient than full sorting when k << n
- Time complexity: O(n log k) vs O(n log n) for full sort
- Space complexity: O(k) for the heap

### Pattern Identification
Look for this pattern when you see:
- "Find the k largest/smallest elements"
- "K most frequent elements"
- "K closest points"
- "Kth largest element"
- "Top k frequent words"

---

## Example 1: Kth Largest Element in an Array (JavaScript)

### Problem
Given an integer array `nums` and an integer `k`, return the `k`th largest element in the array. Note that it is the `k`th largest element in sorted order, not the `k`th distinct element.

**LeetCode**: [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

### Solution

```javascript
/**
 * Find kth largest element using Min-Heap
 * @param {number[]} nums - Array of integers
 * @param {number} k - Position of largest element to find
 * @return {number} - Kth largest element
 */
function findKthLargest(nums, k) {
    // Min-Heap implementation using array
    // JavaScript doesn't have built-in heap, so we'll implement one
    class MinHeap {
        constructor() {
            this.heap = [];
        }

        size() {
            return this.heap.length;
        }

        peek() {
            return this.heap[0];
        }

        push(val) {
            this.heap.push(val);
            this.bubbleUp(this.heap.length - 1);
        }

        pop() {
            if (this.size() === 0) return null;
            if (this.size() === 1) return this.heap.pop();

            const min = this.heap[0];
            this.heap[0] = this.heap.pop();
            this.bubbleDown(0);
            return min;
        }

        bubbleUp(index) {
            while (index > 0) {
                const parentIndex = Math.floor((index - 1) / 2);
                if (this.heap[parentIndex] <= this.heap[index]) break;

                [this.heap[parentIndex], this.heap[index]] =
                    [this.heap[index], this.heap[parentIndex]];
                index = parentIndex;
            }
        }

        bubbleDown(index) {
            while (true) {
                let smallest = index;
                const left = 2 * index + 1;
                const right = 2 * index + 2;

                if (left < this.size() && this.heap[left] < this.heap[smallest]) {
                    smallest = left;
                }
                if (right < this.size() && this.heap[right] < this.heap[smallest]) {
                    smallest = right;
                }
                if (smallest === index) break;

                [this.heap[index], this.heap[smallest]] =
                    [this.heap[smallest], this.heap[index]];
                index = smallest;
            }
        }
    }

    // Use min-heap of size k to find k largest elements
    const minHeap = new MinHeap();

    for (const num of nums) {
        minHeap.push(num);

        // Keep heap size at k
        if (minHeap.size() > k) {
            minHeap.pop();  // Remove smallest
        }
    }

    // Root of min-heap is the kth largest element
    return minHeap.peek();
}

// Alternative solution using built-in sort (simpler but less efficient)
function findKthLargestSort(nums, k) {
    nums.sort((a, b) => b - a);  // Sort descending
    return nums[k - 1];
}

// Example usage
console.log(findKthLargest([3, 2, 1, 5, 6, 4], 2));     // Output: 5
// Explanation: Sorted array is [6, 5, 4, 3, 2, 1], 2nd largest is 5

console.log(findKthLargest([3, 2, 3, 1, 2, 4, 5, 5, 6], 4));  // Output: 4
// Explanation: Sorted array is [6, 5, 5, 4, 3, 3, 2, 2, 1], 4th largest is 4

console.log(findKthLargest([1], 1));                    // Output: 1

console.log(findKthLargest([7, 6, 5, 4, 3, 2, 1], 5));  // Output: 3
```

### Explanation

**Why Min-Heap for k largest?**

- We want to keep track of the k largest elements
- Min-heap keeps smallest element at root
- If heap size > k, we remove the smallest (which is not in top k)
- After processing all elements, heap contains k largest elements
- Root of min-heap is the smallest among k largest = kth largest overall

**Visual Example** for `nums = [3, 2, 1, 5, 6, 4], k = 2`:

```
Process 3: heap = [3]
Process 2: heap = [2, 3]
Process 1: heap = [2, 3], size > k, remove 1, heap = [2, 3]
Process 5: heap = [2, 3, 5], size > k, remove 2, heap = [3, 5]
Process 6: heap = [3, 5, 6], size > k, remove 3, heap = [5, 6]
Process 4: heap = [4, 6, 5], size > k, remove 4, heap = [5, 6]

Final heap = [5, 6] → kth largest (k=2) = 5
```

---

## Example 2: Top K Frequent Elements (Python)

### Problem
Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. You may return the answer in any order.

**LeetCode**: [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

### Solution

```python
from typing import List
from collections import Counter
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        """
        Find k most frequent elements using Min-Heap

        Args:
            nums: Array of integers
            k: Number of most frequent elements to return

        Returns:
            List of k most frequent elements
        """
        # Step 1: Count frequency of each element
        freq_map = Counter(nums)

        # Step 2: Use min-heap to keep track of k most frequent elements
        # Heap stores tuples: (frequency, number)
        min_heap = []

        for num, freq in freq_map.items():
            heapq.heappush(min_heap, (freq, num))

            # Keep heap size at k
            if len(min_heap) > k:
                heapq.heappop(min_heap)  # Remove least frequent

        # Step 3: Extract elements from heap
        result = [num for freq, num in min_heap]

        return result

    def topKFrequentBucketSort(self, nums: List[int], k: int) -> List[int]:
        """
        Alternative solution using bucket sort - O(n) time

        Args:
            nums: Array of integers
            k: Number of most frequent elements to return

        Returns:
            List of k most frequent elements
        """
        # Count frequencies
        freq_map = Counter(nums)

        # Create buckets where index represents frequency
        # bucket[i] contains all numbers with frequency i
        buckets = [[] for _ in range(len(nums) + 1)]

        for num, freq in freq_map.items():
            buckets[freq].append(num)

        # Collect k most frequent elements from buckets (right to left)
        result = []
        for i in range(len(buckets) - 1, 0, -1):
            for num in buckets[i]:
                result.append(num)
                if len(result) == k:
                    return result

        return result

# Example usage
solution = Solution()

# Example 1
nums1 = [1, 1, 1, 2, 2, 3]
k1 = 2
print(solution.topKFrequent(nums1, k1))  # Output: [1, 2]
# Explanation: 1 appears 3 times, 2 appears 2 times (most frequent)

# Example 2
nums2 = [1]
k2 = 1
print(solution.topKFrequent(nums2, k2))  # Output: [1]

# Example 3
nums3 = [4, 1, -1, 2, -1, 2, 3]
k3 = 2
print(solution.topKFrequent(nums3, k3))  # Output: [-1, 2] (or [2, -1])
# Explanation: -1 appears 2 times, 2 appears 2 times

# Example 4 - Using bucket sort
nums4 = [1, 1, 1, 2, 2, 3]
k4 = 2
print(solution.topKFrequentBucketSort(nums4, k4))  # Output: [1, 2]
```

### Explanation

**Heap-based Approach**:

1. **Count frequencies**: Use hashmap to count occurrences
2. **Build min-heap**: Store (frequency, element) pairs
3. **Maintain size k**: If heap exceeds k, remove least frequent
4. **Extract result**: Elements in heap are k most frequent

**Visual Example** for `nums = [1,1,1,2,2,3], k = 2`:

```
Frequency map: {1: 3, 2: 2, 3: 1}

Process (3, 1): heap = [(3, 1)]
Process (2, 2): heap = [(2, 2), (3, 1)]
Process (1, 3): heap = [(1, 3), (3, 1), (2, 2)]
                size > k, remove (1, 3)
                heap = [(2, 2), (3, 1)]

Result: [2, 1] (extract elements from heap)
```

**Bucket Sort Approach** (More efficient for this problem):

```
Frequency map: {1: 3, 2: 2, 3: 1}

Buckets:
  bucket[0] = []
  bucket[1] = [3]     (3 appears 1 time)
  bucket[2] = [2]     (2 appears 2 times)
  bucket[3] = [1]     (1 appears 3 times)

Iterate from right: [1] → [2] → k elements collected
Result: [1, 2]
```

---

## Time & Space Complexity

### Example 1: Kth Largest Element
- **Heap approach**:
  - Time Complexity: O(n log k) - Process n elements, each heap operation is O(log k)
  - Space Complexity: O(k) - Heap size
- **Sort approach**:
  - Time Complexity: O(n log n)
  - Space Complexity: O(1) or O(n) depending on sort implementation

### Example 2: Top K Frequent
- **Heap approach**:
  - Time Complexity: O(n log k) - n for counting, n log k for heap operations
  - Space Complexity: O(n) - HashMap + O(k) heap
- **Bucket sort approach**:
  - Time Complexity: O(n)
  - Space Complexity: O(n)

---

## Common Variations

1. **Kth Largest/Smallest Element**
   - LeetCode: [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
   - LeetCode: [703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

2. **Top K Frequent Elements**
   - LeetCode: [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
   - LeetCode: [692. Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/)

3. **K Closest Points**
   - LeetCode: [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

4. **K Pairs with Smallest Sums**
   - LeetCode: [373. Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/)

---

## Practice Problems

### Easy
1. [703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

### Medium
2. [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
3. [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
4. [692. Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/)
5. [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)
6. [658. Find K Closest Elements](https://leetcode.com/problems/find-k-closest-elements/)
7. [767. Reorganize String](https://leetcode.com/problems/reorganize-string/)

### Hard
8. [295. Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)
9. [502. IPO](https://leetcode.com/problems/ipo/)

---

## Key Takeaways

1. **Heap choice**:
   - Min-heap for k largest elements (keep largest k, remove smallest)
   - Max-heap for k smallest elements (keep smallest k, remove largest)

2. **Efficiency**: O(n log k) vs O(n log n) - significant when k << n

3. **Heap size**: Always maintain heap at size k

4. **Python heapq**: Python's heapq is a min-heap by default
   - For max-heap, negate values or use custom comparator

5. **Alternative approaches**: Quickselect (O(n) average), Bucket sort (for frequency problems)

6. **Streaming data**: Particularly useful when data arrives in streams

[← Previous: Monotonic Stack](./06-monotonic-stack.md) | [Back to Index](./README.md) | [Next: Overlapping Intervals →](./08-overlapping-intervals.md)
