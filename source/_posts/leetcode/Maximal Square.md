---
title: Maximal Rectangle
categories:
  - leetcode
tags:
  - hard
  - DP
---

# Maximal Square #medium

Given an `m x n` binary `matrix` filled with `0`'s and `1`'s, _find the largest square containing only_ `1`'s _and return its area_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/26/max1grid.jpg)

**Input:** matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
**Output:** 4

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/11/26/max2grid.jpg)

**Input:** matrix = [["0","1"],["1","0"]]
**Output:** 1

**Example 3:**

**Input:** matrix = [["0"]]
**Output:** 0

**Constraints:**

-   `m == matrix.length`
-   `n == matrix[i].length`
-   `1 <= m, n <= 300`
-   `matrix[i][j]` is `'0'` or `'1'`.

# Solution

### DP

直接用 **DP**  O(mn)/O(mn) 
==space can be minimized to O(n)，因为每次只考虑相邻两行==

每个 `dp[i,j]` 都记录当前形成的最大的 square 的 area 记录在表里面，
然后我们 从左往右，从上往下 遍历

对于 `dp[i,j]`，
1. 如果 `matrix[i,j] == 1` 我们通过画图可以得到 
	`dp[i,j] = min(dp[i-1,j], dp[i-1,j-1], dp[i,j-1]) + 1`
	其余的就 ignore。

```java
public class Solution {
    public int maximalSquare(char[][] matrix) {
        int rows = matrix.length, cols = rows > 0 ? matrix[0].length : 0;
        int[] dp = new int[cols + 1];
        int maxsqlen = 0, prev = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                int temp = dp[j];
                if (matrix[i - 1][j - 1] == '1') {
                    dp[j] = Math.min(Math.min(dp[j - 1], prev), dp[j]) + 1;
                    maxsqlen = Math.max(maxsqlen, dp[j]);
                } else {
                    dp[j] = 0;
                }
                prev = temp;
            }
        }
        return maxsqlen * maxsqlen;
    }
}
```