# Dynamic Programming Pattern

## Pattern Overview

**Dynamic Programming (DP)** solves complex problems by breaking them into simpler overlapping subproblems and storing results to avoid redundant computation. It combines optimal substructure with memoization or tabulation.

### When to Use
- Optimization problems (min/max)
- Counting problems
- Decision problems (can/cannot)
- Problems with overlapping subproblems
- Finding longest/shortest/most optimal solution

### Key Characteristics
- Overlapping subproblems
- Optimal substructure
- Two approaches: Top-down (memoization) or Bottom-up (tabulation)
- Trades space for time
- Often involves 1D or 2D arrays/tables

### Pattern Identification
Look for this pattern when you see:
- "Find maximum/minimum..."
- "Count number of ways..."
- "Longest/shortest..."
- "Can you reach..."
- Fibonacci-like recurrence relations
- "Optimal" or "best" solution

---

## Example 1: Climbing Stairs (JavaScript)

### Problem
You are climbing a staircase. It takes `n` steps to reach the top. Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

**LeetCode**: [70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)

### Solution

```javascript
/**
 * Count ways to climb stairs - Top-down (Memoization)
 * @param {number} n - Number of stairs
 * @return {number} - Number of distinct ways
 */
function climbStairs(n) {
    const memo = new Map();

    function dp(i) {
        // Base cases
        if (i <= 2) return i;

        // Check memo
        if (memo.has(i)) return memo.get(i);

        // Recurrence: ways(i) = ways(i-1) + ways(i-2)
        const result = dp(i - 1) + dp(i - 2);

        // Store in memo
        memo.set(i, result);
        return result;
    }

    return dp(n);
}

/**
 * Bottom-up approach (Tabulation)
 * @param {number} n
 * @return {number}
 */
function climbStairsBottomUp(n) {
    if (n <= 2) return n;

    // dp[i] = number of ways to reach step i
    const dp = new Array(n + 1);
    dp[1] = 1;  // 1 way to reach step 1
    dp[2] = 2;  // 2 ways to reach step 2

    // Build up from smaller subproblems
    for (let i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[n];
}

/**
 * Space-optimized approach
 * @param {number} n
 * @return {number}
 */
function climbStairsOptimized(n) {
    if (n <= 2) return n;

    let prev2 = 1;  // dp[i-2]
    let prev1 = 2;  // dp[i-1]

    for (let i = 3; i <= n; i++) {
        const current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}

// Example usage
console.log(climbStairs(2));           // Output: 2
// Explanation: 1+1 or 2

console.log(climbStairs(3));           // Output: 3
// Explanation: 1+1+1, 1+2, or 2+1

console.log(climbStairsBottomUp(5));   // Output: 8
// Explanation: 1+1+1+1+1, 1+1+1+2, 1+1+2+1, 1+2+1+1, 2+1+1+1,
//              1+2+2, 2+1+2, 2+2+1

console.log(climbStairsOptimized(4));  // Output: 5
```

### Explanation

**Recurrence Relation**:
```
dp[i] = dp[i-1] + dp[i-2]

Why? To reach step i, you can:
- Take 1 step from step i-1 (dp[i-1] ways)
- Take 2 steps from step i-2 (dp[i-2] ways)
```

**Visual Example for n=5**:
```
Step 0: 0 ways (base)
Step 1: 1 way  [1]
Step 2: 2 ways [1+1, 2]
Step 3: 3 ways [1+1+1, 1+2, 2+1]
Step 4: 5 ways [1+1+1+1, 1+1+2, 1+2+1, 2+1+1, 2+2]
Step 5: 8 ways [combine step 3 + step 4]

dp[3] = dp[2] + dp[1] = 2 + 1 = 3
dp[4] = dp[3] + dp[2] = 3 + 2 = 5
dp[5] = dp[4] + dp[3] = 5 + 3 = 8
```

---

## Example 2: Coin Change (Python)

### Problem
You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money. Return the fewest number of coins needed to make up that amount. If impossible, return -1.

**LeetCode**: [322. Coin Change](https://leetcode.com/problems/coin-change/)

### Solution

```python
from typing import List

class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        """
        Find minimum coins needed using bottom-up DP

        Args:
            coins: List of coin denominations
            amount: Target amount

        Returns:
            Minimum number of coins, or -1 if impossible
        """
        # dp[i] = minimum coins needed to make amount i
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0  # 0 coins needed for amount 0

        # For each amount from 1 to target
        for i in range(1, amount + 1):
            # Try each coin
            for coin in coins:
                if coin <= i:
                    # Take coin and add to solution for (i - coin)
                    dp[i] = min(dp[i], dp[i - coin] + 1)

        # Return result, or -1 if impossible
        return dp[amount] if dp[amount] != float('inf') else -1

    def coinChangeTopDown(self, coins: List[int], amount: int) -> int:
        """
        Alternative: Top-down with memoization
        """
        memo = {}

        def dp(remaining):
            # Base cases
            if remaining == 0:
                return 0
            if remaining < 0:
                return float('inf')

            # Check memo
            if remaining in memo:
                return memo[remaining]

            # Try each coin and take minimum
            min_coins = float('inf')
            for coin in coins:
                result = dp(remaining - coin)
                if result != float('inf'):
                    min_coins = min(min_coins, result + 1)

            memo[remaining] = min_coins
            return min_coins

        result = dp(amount)
        return result if result != float('inf') else -1

# Example usage
solution = Solution()

# Example 1
coins1 = [1, 2, 5]
amount1 = 11
print(solution.coinChange(coins1, amount1))  # Output: 3
# Explanation: 11 = 5 + 5 + 1 (3 coins)

# Example 2
coins2 = [2]
amount2 = 3
print(solution.coinChange(coins2, amount2))  # Output: -1
# Explanation: Cannot make 3 with only coin of 2

# Example 3
coins3 = [1]
amount3 = 0
print(solution.coinChange(coins3, amount3))  # Output: 0

# Example 4 - Using top-down approach
coins4 = [1, 2, 5]
amount4 = 11
print(solution.coinChangeTopDown(coins4, amount4))  # Output: 3
```

### Explanation

**Recurrence Relation**:
```
dp[i] = min(dp[i], dp[i - coin] + 1) for each coin

Why? To make amount i:
- Try each coin denomination
- If coin <= i, take it and solve for (i - coin)
- Choose the coin that gives minimum total coins
```

**DP Table for coins=[1,2,5], amount=11**:
```
Amount:  0  1  2  3  4  5  6  7  8  9  10 11
dp[i]:   0  1  1  2  2  1  2  2  3  3  2  3

Explanation:
dp[0] = 0 (base case)
dp[1] = 1 (1×1)
dp[2] = 1 (1×2)
dp[3] = 2 (1×2 + 1×1)
dp[4] = 2 (2×2)
dp[5] = 1 (1×5)
dp[6] = 2 (1×5 + 1×1)
...
dp[11] = 3 (2×5 + 1×1)
```

**Step-by-step for amount=11**:
```
For i=11, try each coin:

Coin=1: dp[11] = min(∞, dp[10] + 1) = min(∞, 2+1) = 3
Coin=2: dp[11] = min(3, dp[9] + 1) = min(3, 3+1) = 3
Coin=5: dp[11] = min(3, dp[6] + 1) = min(3, 2+1) = 3

Final: dp[11] = 3
```

---

## Time & Space Complexity

### Example 1: Climbing Stairs
- **Top-down with memo**:
  - Time: O(n) - Each state computed once
  - Space: O(n) - Memoization + recursion stack
- **Bottom-up**:
  - Time: O(n)
  - Space: O(n) - DP array
- **Optimized**:
  - Time: O(n)
  - Space: O(1) - Only two variables

### Example 2: Coin Change
- **Time Complexity**: O(amount × coins.length)
- **Space Complexity**: O(amount) - DP array

---

## Common Variations

1. **Fibonacci-style**
   - LeetCode: [70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)
   - LeetCode: [509. Fibonacci Number](https://leetcode.com/problems/fibonacci-number/)

2. **Knapsack Problems**
   - LeetCode: [322. Coin Change](https://leetcode.com/problems/coin-change/)
   - LeetCode: [518. Coin Change II](https://leetcode.com/problems/coin-change-ii/)
   - LeetCode: [416. Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

3. **Longest/Shortest Subsequence**
   - LeetCode: [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)
   - LeetCode: [1143. Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

4. **Grid-based DP**
   - LeetCode: [62. Unique Paths](https://leetcode.com/problems/unique-paths/)
   - LeetCode: [64. Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)

5. **String DP**
   - LeetCode: [72. Edit Distance](https://leetcode.com/problems/edit-distance/)
   - LeetCode: [139. Word Break](https://leetcode.com/problems/word-break/)

---

## Practice Problems

### Easy
1. [70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)
2. [509. Fibonacci Number](https://leetcode.com/problems/fibonacci-number/)
3. [746. Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/)

### Medium
4. [322. Coin Change](https://leetcode.com/problems/coin-change/)
5. [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)
6. [62. Unique Paths](https://leetcode.com/problems/unique-paths/)
7. [198. House Robber](https://leetcode.com/problems/house-robber/)
8. [139. Word Break](https://leetcode.com/problems/word-break/)
9. [416. Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

### Hard
10. [72. Edit Distance](https://leetcode.com/problems/edit-distance/)
11. [1143. Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)
12. [10. Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/)

---

## Key Takeaways

1. **Two approaches**:
   - **Top-down (Memoization)**: Start from problem, recurse, cache results
   - **Bottom-up (Tabulation)**: Start from base cases, build up

2. **Identify DP**:
   - Overlapping subproblems
   - Optimal substructure
   - Optimization/counting problems

3. **Steps to solve**:
   1. Define state (what does dp[i] represent?)
   2. Find recurrence relation
   3. Identify base cases
   4. Determine computation order (bottom-up) or add memoization (top-down)
   5. Space optimization if possible

4. **Space optimization**: Often can reduce from O(n) or O(n²) to O(1) or O(n)

5. **Common patterns**:
   - 1D DP: Previous state(s)
   - 2D DP: Grid or two sequences

6. **Start simple**: Solve recursively first, then add memoization, then consider tabulation

[← Previous: Backtracking](./13-backtracking.md) | [Back to Index](./README.md) | [Next: Graph Algorithms →](./15-graph-algorithms.md)
