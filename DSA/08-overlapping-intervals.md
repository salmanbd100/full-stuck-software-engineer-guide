# Overlapping Intervals Pattern

## Pattern Overview

The **Overlapping Intervals** pattern deals with problems involving intervals (ranges) that may overlap with each other. This pattern typically involves sorting intervals and then merging, inserting, or finding overlaps.

### When to Use
- Merging overlapping intervals
- Finding free time slots or conflicts
- Scheduling problems (meeting rooms)
- Inserting new intervals
- Checking if intervals overlap

### Key Characteristics
- Usually requires sorting intervals first (by start time)
- Compares current interval with previous/next interval
- Often involves merging or counting overlaps
- Time complexity: O(n log n) for sorting + O(n) for processing

### Pattern Identification
Look for this pattern when you see:
- "Merge overlapping intervals"
- "Meeting rooms" or scheduling problems
- "Insert interval"
- "Find free time"
- "Check if intervals conflict"
- Problems involving time ranges, appointments, or schedules

---

## Example 1: Merge Intervals (JavaScript)

### Problem
Given an array of `intervals` where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals and return an array of the non-overlapping intervals.

**LeetCode**: [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)

### Solution

```javascript
/**
 * Merge overlapping intervals
 * @param {number[][]} intervals - Array of intervals [start, end]
 * @return {number[][]} - Merged non-overlapping intervals
 */
function merge(intervals) {
    // Edge case: empty or single interval
    if (intervals.length <= 1) {
        return intervals;
    }

    // Step 1: Sort intervals by start time
    intervals.sort((a, b) => a[0] - b[0]);

    // Step 2: Initialize result with first interval
    const merged = [intervals[0]];

    // Step 3: Iterate through remaining intervals
    for (let i = 1; i < intervals.length; i++) {
        const currentInterval = intervals[i];
        const lastMerged = merged[merged.length - 1];

        // Check if current interval overlaps with last merged interval
        if (currentInterval[0] <= lastMerged[1]) {
            // Overlaps - merge by extending end time
            lastMerged[1] = Math.max(lastMerged[1], currentInterval[1]);
        } else {
            // No overlap - add current interval as new interval
            merged.push(currentInterval);
        }
    }

    return merged;
}

// Example usage
console.log(merge([[1, 3], [2, 6], [8, 10], [15, 18]]));
// Output: [[1, 6], [8, 10], [15, 18]]
// Explanation: [1,3] and [2,6] overlap → merged to [1,6]

console.log(merge([[1, 4], [4, 5]]));
// Output: [[1, 5]]
// Explanation: [1,4] and [4,5] overlap at 4 → merged to [1,5]

console.log(merge([[1, 4], [0, 4]]));
// Output: [[0, 4]]
// Explanation: After sorting → [[0,4], [1,4]] → merged to [0,4]

console.log(merge([[1, 4], [2, 3]]));
// Output: [[1, 4]]
// Explanation: [2,3] is completely inside [1,4] → merged to [1,4]
```

### Explanation

**How to check overlap**:
- Two intervals `[a, b]` and `[c, d]` overlap if `c <= b` (assuming sorted by start)
- When overlapping, merge to `[a, max(b, d)]`

**Visual Example**:
```
Input: [[1,3], [2,6], [8,10], [15,18]]

After sorting: [[1,3], [2,6], [8,10], [15,18]]

Step 1: merged = [[1,3]]

Step 2: [2,6] overlaps with [1,3] (2 <= 3)
        Merge: [1, max(3,6)] = [1,6]
        merged = [[1,6]]

Step 3: [8,10] doesn't overlap with [1,6] (8 > 6)
        Add new interval
        merged = [[1,6], [8,10]]

Step 4: [15,18] doesn't overlap with [8,10] (15 > 10)
        Add new interval
        merged = [[1,6], [8,10], [15,18]]

Result: [[1,6], [8,10], [15,18]]
```

**Edge Cases**:
1. Completely contained: `[1,4]` contains `[2,3]` → merge to `[1,4]`
2. Touching at boundary: `[1,4]` and `[4,5]` → merge to `[1,5]`
3. No overlap: `[1,3]` and `[4,6]` → keep both

---

## Example 2: Meeting Rooms II (Python)

### Problem
Given an array of meeting time intervals consisting of start and end times `[[s1,e1],[s2,e2],...]`, find the minimum number of conference rooms required.

**LeetCode**: [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) (Premium)

### Solution

```python
from typing import List
import heapq

class Solution:
    def minMeetingRooms(self, intervals: List[List[int]]) -> int:
        """
        Find minimum number of meeting rooms required using heap

        Args:
            intervals: List of meeting intervals [start, end]

        Returns:
            Minimum number of meeting rooms needed
        """
        # Edge case
        if not intervals:
            return 0

        # Step 1: Sort meetings by start time
        intervals.sort(key=lambda x: x[0])

        # Step 2: Use min-heap to track end times of ongoing meetings
        # Heap stores end times of meetings currently in progress
        meeting_rooms = []

        # Step 3: Process each meeting
        for interval in intervals:
            start, end = interval

            # If earliest ending meeting finishes before current starts,
            # we can reuse that room
            if meeting_rooms and meeting_rooms[0] <= start:
                heapq.heappop(meeting_rooms)  # Remove finished meeting

            # Add current meeting's end time to heap
            heapq.heappush(meeting_rooms, end)

        # Number of rooms needed = size of heap (concurrent meetings)
        return len(meeting_rooms)

    def minMeetingRoomsAlternative(self, intervals: List[List[int]]) -> int:
        """
        Alternative solution using separate start/end arrays

        Time: O(n log n), Space: O(n)
        """
        if not intervals:
            return 0

        # Separate start and end times
        start_times = sorted([i[0] for i in intervals])
        end_times = sorted([i[1] for i in intervals])

        rooms_needed = 0
        max_rooms = 0
        start_ptr = 0
        end_ptr = 0

        # Process all start times
        while start_ptr < len(start_times):
            # If a meeting starts before earliest one ends
            if start_times[start_ptr] < end_times[end_ptr]:
                rooms_needed += 1
                max_rooms = max(max_rooms, rooms_needed)
                start_ptr += 1
            else:
                # A meeting ended, free up a room
                rooms_needed -= 1
                end_ptr += 1

        return max_rooms

# Example usage
solution = Solution()

# Example 1
intervals1 = [[0, 30], [5, 10], [15, 20]]
print(solution.minMeetingRooms(intervals1))  # Output: 2
# Explanation:
# Meeting 1: 0-30
# Meeting 2: 5-10 (overlaps with 1, need room 2)
# Meeting 3: 15-20 (still overlaps with 1, can reuse room 2)
# Max concurrent meetings: 2

# Example 2
intervals2 = [[7, 10], [2, 4]]
print(solution.minMeetingRooms(intervals2))  # Output: 1
# Explanation: Meetings don't overlap, can use same room

# Example 3
intervals3 = [[1, 5], [2, 3], [4, 6], [5, 7]]
print(solution.minMeetingRooms(intervals3))  # Output: 3
# Explanation:
# Time 2-3: [1,5], [2,3] overlap → 2 rooms
# Time 4-5: [1,5], [4,6] overlap → 2 rooms
# Time 5-6: [1,5], [4,6], [5,7] overlap → 3 rooms needed

# Example 4
intervals4 = [[0, 30], [5, 10], [15, 20]]
print(solution.minMeetingRoomsAlternative(intervals4))  # Output: 2
```

### Explanation

**Heap-based Approach**:

1. **Sort by start time**: Process meetings in chronological order
2. **Min-heap of end times**: Track when current meetings will end
3. **For each meeting**:
   - Check if earliest ending meeting finishes before current starts
   - If yes, reuse that room (pop from heap)
   - Add current meeting's end time to heap
4. **Result**: Heap size = number of concurrent meetings = rooms needed

**Visual Example** for `[[0,30], [5,10], [15,20]]`:

```
After sorting: [[0,30], [5,10], [15,20]]

Meeting [0,30]:
  heap = []
  No meeting ends before 0
  Push 30
  heap = [30]
  max_rooms = 1

Meeting [5,10]:
  heap = [30]
  Earliest end is 30, but 30 > 5 (meeting not finished)
  Can't reuse room
  Push 10
  heap = [10, 30]  (min-heap property)
  max_rooms = 2

Meeting [15,20]:
  heap = [10, 30]
  Earliest end is 10, and 10 <= 15 (meeting finished!)
  Reuse room, pop 10
  heap = [30]
  Push 20
  heap = [20, 30]
  max_rooms = 2

Result: 2 rooms needed
```

---

## Time & Space Complexity

### Example 1: Merge Intervals
- **Time Complexity**: O(n log n) - Sorting dominates
- **Space Complexity**: O(n) - Result array (or O(log n) for sorting)

### Example 2: Meeting Rooms II
- **Heap approach**:
  - Time Complexity: O(n log n) - Sorting + n heap operations
  - Space Complexity: O(n) - Heap can contain all meetings
- **Two arrays approach**:
  - Time Complexity: O(n log n) - Sorting
  - Space Complexity: O(n) - Two arrays

---

## Common Variations

1. **Merge Intervals**
   - Basic merging of overlapping intervals
   - LeetCode: [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)

2. **Insert Interval**
   - Insert new interval and merge if necessary
   - LeetCode: [57. Insert Interval](https://leetcode.com/problems/insert-interval/)

3. **Meeting Rooms**
   - Check if person can attend all meetings
   - LeetCode: [252. Meeting Rooms](https://leetcode.com/problems/meeting-rooms/)

4. **Meeting Rooms II**
   - Find minimum conference rooms needed
   - LeetCode: [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)

5. **Non-overlapping Intervals**
   - Find minimum intervals to remove to make non-overlapping
   - LeetCode: [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)

6. **Employee Free Time**
   - Find free time intervals for all employees
   - LeetCode: [759. Employee Free Time](https://leetcode.com/problems/employee-free-time/)

---

## Practice Problems

### Easy
1. [252. Meeting Rooms](https://leetcode.com/problems/meeting-rooms/)

### Medium
2. [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)
3. [57. Insert Interval](https://leetcode.com/problems/insert-interval/)
4. [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)
5. [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)
6. [452. Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)
7. [986. Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/)

### Hard
8. [759. Employee Free Time](https://leetcode.com/problems/employee-free-time/)

---

## Key Takeaways

1. **Always sort first**: Sort by start time (or end time depending on problem)

2. **Overlap condition**: Two intervals `[a,b]` and `[c,d]` overlap if `c <= b` (when sorted by start)

3. **Merge formula**: When merging `[a,b]` and `[c,d]`, result is `[a, max(b,d)]`

4. **Common patterns**:
   - Merging: Track last merged interval
   - Counting overlaps: Use heap or sweep line algorithm
   - Finding gaps: Look for spaces between merged intervals

5. **Edge cases**: Touching intervals `[1,2]` and `[2,3]`, contained intervals `[1,5]` and `[2,3]`

6. **Heap for scheduling**: Min-heap of end times tracks active intervals

[← Previous: Top K Elements](./07-top-k-elements.md) | [Back to Index](./README.md) | [Next: Modified Binary Search →](./09-modified-binary-search.md)
