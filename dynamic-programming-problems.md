# Dynamic Programming - Optimisation through Overlapping Subproblems

## How to think clearly about coin DP Problems

### Whats the goal? Count or Minimise?
- if counting combinations -> use `+=`
- if minimising coins -> use `min`

### Does Order Matter?
- **if order matters** (permutations) -> loop over **amounts first**, then coins
- **if order doesn't matter** (combinations) -> loop over **coins first**

### What's the base case?
For counting ways:
  - `dp[0] = 1` -> One way to make 0 (do nothing)

For min coins:
  - `dp[0]` = 0 -> 0 coins to make amount 0

### How does each `dp[i]` relate to smaller subproblems
In counting:
 > dp[i] += dp[i - coin]

 - We're adding the number of ways to make the smaller amount

In min coins:
> dp[i] = min(dp[i], 1 + dp[i - coin])

- We're **choosing** the cheaper option

### Visualisation tip
> Each `dp[i]` is the answer to "what's the total count (or min coins) to make amount `i`"


## Coin Change: Counting Ways

### Problem Summary
Input:
- `coins[]` -> available denominations
- `amount` -> total target amount

Output:
- Number of **distinct combinations** to form the amount (order does not matter)

### Core Pattern
Use a 1D DP array where `dp[i]` = number of ways to make amount `i`.

```java
public int countWays(int[] coins, int amount) {
  int[] dp = new int[amount + 1];
  dp[0] = 1; // base case: 1 way to make 0

  for (int coin : coins) {
    for (int i = coin; i <= amount; i++) {
      dp[i] += dp[i - coin];
    }
  }

  return dp[amount];
}
```

### Common Pitfalls
- Using `dp[0] = 0` -> no combinations ever form
- looping amounts before coins -> causes overcounting
- not understand what `dp[i - coin]` represents (ways to build up to `i`)

### Transferable Takeaway
When combinations are required from unlimited supplies of items (e.g. coins) and **order doesn't matter**, use:
- 1D bottom-up DP
- Outer loop over items
- Accumulate from subproblems


## Coin Change - Minimum Number of Coins

### Problem Summary
Input:
  - `coins[]` - list of available denominations
  - `amount` - total value to construct

Goal:
  - return the minimum number of coins to make that amount
  - if it's not possible, return `-1`

### Core Idea
> Use **1D DP** where `dp[i]` represents the **minimum coins** needed to form amount `i`

- Bottom-up: build up from smaller amounts (dp[0])

```java
public int coinChange(int[] coins, int amount) {
  int[] dp = new int[amount - 1];
  Arrays.fill(dp, amount + 1); // acts like infinity
  dp[0] = 0; // base case

  for (int coin : coins) {
    for (int i = coin; i <= amount; i++) {
      dp[i] = Math.min(dp[i], 1 + dp[i - coin]);
    }
  }

  return dp[amount] > amount ? -1 : dp[amount];
}
```

### Common Pitfalls
- using `+=` instead of `min()` - that counts combinations instead of minimising
- forgetting to initialise with `amount + 1` - makes invalid results look valid
- looping in the wrong order (e.g. `i = 0 to amount`) - use `i = coin`

### Transferable Takeaway
This pattern is perfect for `minimisation problems` with `unlimited supply of elements`:
- Rod cutting
- Knapsack
- Word segmentation



## Longest Common Subsequence (LCS)

### Problem Summary
Input:
  - Two strings `text1` and `text2`

Output:
  - Length of their longest common subsequence (LCS)

### Example
```sh
text1 = "abcde", text2 = "ace"
LCS = "ace" -> length = 3
```

### Core DP Idea
> Use a **2D DP** table where `dp[i][j]` = length of LCS between:

- `text1[0..i-1]`
- `text1[0..j-1]`

#### Transition Logic
If characters match:
```java
dp[i][j] = 1 + dp[i - 1][j - 1];
```

If not:
```java
dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
```

#### Base Case
```java
dp[0][*] = 0 // comparing "" to prefix of text2
dp[*][0] = 0 // comparing "" to prefix of text1
```

#### Final Return Value
```java
dp[m][n] // full LCS between text1 and text2
```

### Java Code
```java
public int longestCommonSubsequence(String text1, String text2) {
  int m = text1.length();
  int n = text2.length();
  int[][] dp = new int[m + 1][n + 1];

  for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
      if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
        dp[i][j] = 1 + dp[i - 1][j - 1];
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }

  return dp[m][n];
}
```

### Common Pitfalls
- Off-by-one errors in indices
- Comparing `i` and `j` directly (must use `charAt(i - 1)`)
- Forgetting to initialise base cases (`dp[0][*] = 0`)

### Transferable Takeaway
> LCS is the standard template for comparing two sequences. It builds a **DP grid** from pairwise character comparisons.

Use this structure for:
  - Diff tools
  - DNA alignment
  - String similarity problems (Edit Distance, etc)



## 0/1 Knapsack Problem

### Problem Summary
Input:
  - `weights[]`: weight of each item
  - `values[]`: value of each item
  - `capacity`: max weight the knapsack can hold

Goal:
  - Maximise total value of selected items
  - Can **take each item at most once**
  - Cannot exceed capacity


### DP State Definition
`dp[i][w]` = max value using first i items with with knapsack capacity w

### Transition Logic
if item `i - 1` fits in capacity `w`:
```java
dp[i][w] = Math.max(
  dp[i - 1][w] // exclude item
  dp[i - w][w - weights[i - 1]] + values[i - 1] // include item
);
```

If item doesn't fit:
```java
dp[i][w] = dp[i - 1][w];
```

### Base Case
```java
dp[0][*] = 0 // 0 items = 0 value
dp[*][0] = 0 // 0 capacity = 0 value
```

### Final Answer
```java
dp[n][capacity]
```
where n = `weights.length`

### Java Implementation
```java
public int knapsack(int[] weights, int[] values, int capacity) {
  int n = weights.length;
  int[][]dp = new int[n + 1][capacity + 1];

  for (int i = 1; i <= n; i++) {
    for (int w = 0; w <= capacity; w++) {
      if (weights[i - 1] <= w) {
        dp[i][w] = Math.max(dp[i - 1][w],
                            dp[i - 1][w - weights[i - 1]] + value);
      } else {
        dp[i][w] = dp[i - 1][w];
      }
    }
  }

  return dp[n][capacity];
}
```

### Common Pitfalls
- Using `weights[i]` instead of `weights[i - 1]`
- Forgetting to check `if weights[i - 1] <= w`
- Off-by-one when initialising or returning result

### Transferable Takeaway
> Use this 2D DP pattern for **resource-constrained optimisation**, where:

- each item has a cost (weight)
- you can only pick once
- you want to **maximise** or **minimise** something

Also builds intuition for:
- Subset sum
- Partition problems
- Advanced versions like Unbounded Knapsack, Multi-Knapsack etc.


