---
title: Maximal Rectangle
categories:
  - leetcode
tags:
  - hard
  - stack
  - DP
---

# Maximal Rectangle #hard

 [Maximal Square.md](Maximal Square.md) 

 [Largest Rectangle in Histogram.md](Largest Rectangle in Histogram.md) 

Given a `rows x cols` binary `matrix` filled with `0`'s and `1`'s, find the largest rectangle containing only `1`'s and return _its area_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/09/14/maximal.jpg)

**Input:** matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
**Output:** 6
**Explanation:** The maximal rectangle is shown in the above picture.

**Example 2:**

**Input:** matrix = []
**Output:** 0

**Example 3:**

**Input:** matrix = [["0"]]
**Output:** 0

**Example 4:**

**Input:** matrix = [["1"]]
**Output:** 1

**Example 5:**

**Input:** matrix = [["0","0"]]
**Output:** 0

**Constraints:**

-   `rows == matrix.length`
-   `cols == matrix[i].length`
-   `0 <= row, cols <= 200`
-   `matrix[i][j]` is `'0'` or `'1'`.


# Solution:

### DP solution:

注意这里和 maximal square 的不同，这里 DP 不能很好地显示 diagonal 的情况，**O(mn^2)/O(mn)**

考虑 `dp[i,j]` 记录下当前 row 的到这里为止的 最长的 rect width
之后 寻找 maximal area 的时候，

我们从 `0...i-1` 看看，所能达到的最大的 area 是多少。重复这个步骤

```python
 def maximalRectangle(self, matrix: List[List[str]]) -> int:
     maxarea = 0

     dp = [[0] * len(matrix[0]) for _ in range(len(matrix))]
     for i in range(len(matrix)):
         for j in range(len(matrix[0])):
             if matrix[i][j] == '0': continue

             # compute the maximum width and update dp with it
             width = dp[i][j] = dp[i][j-1] + 1 if j else 1

             # compute the maximum area rectangle with a lower right corner at [i, j]
             for k in range(i, -1, -1):
                 width = min(width, dp[k][j])
                 maxarea = max(maxarea, width * (i-k+1))
     return maxarea
```



### ==Histograms==

 [Largest Rectangle in Histogram.md](Largest Rectangle in Histogram.md) 

**O(mn)/O(n)**

依照上面的方法得到 `dp`，每一个 column 往上都是一个 histogram，histogram 找 largest 需要 O(n), 所以总共 需要 O(2mn) 

- 计算右边的 histograms 就直接加减就行

```python
# Get the maximum area in a histogram given its heights
 def leetcode84(self, heights):
     stack = [-1]

     maxarea = 0
     for i in range(len(heights)):

         while stack[-1] != -1 and heights[stack[-1]] >= heights[i]:
             maxarea = max(maxarea, heights[stack.pop()] * (i - stack[-1] - 1))
         stack.append(i)

     while stack[-1] != -1:
         maxarea = max(maxarea, heights[stack.pop()] * (len(heights) - stack[-1] - 1))
     return maxarea
```


    def maximalRectangle(self, matrix: List[List[str]]) -> int:
    
        if not matrix: return 0
    
        maxarea = 0
        dp = [0] * len(matrix[0])
        for i in range(len(matrix)):
            for j in range(len(matrix[0])):
    
                # update the state of this row's histogram using the last row's histogram
                # by keeping track of the number of consecutive ones
    
                dp[j] = dp[j] + 1 if matrix[i][j] == '1' else 0
    
            # update maxarea with the maximum area from this row's histogram
            maxarea = max(maxarea, self.leetcode84(dp))
        return maxarea
    
    ```



### ==DP solution better==

**O(mn)/O(m)**

idea 就是构建三个 DP table of size m x n:

1. heights: 记录当前 slot 往上最高的 height
2. left: 记录当前 slot 的高度能延伸到的最左边的 index
3. right: 记录当前 slot 的高度能延伸到的最右边的 index

**heights init 成 0，left init 成 0，right init 成 n <u>(#column)</u>**

==**Update Rule (都只 apply to slot with value = 1):**==

1. `height[i+1][j] = height[i][j] + 1  `
2. `left[i+1][j] = max(left[i][j], current_left) `
3. `right[i+1][j] = min(right[i][j], current_right)`

> current_left 就是 当前 row 能到的最左边
>
> current_right 就是当前 row 能到的最右边

下面展示的代码是 把上面的 space 优化的结果，因为 DP 只联系了相邻两层，所以只需要 **n-sized vector** 即可


```python
def maximalRectangle(self, matrix: List[List[str]]) -> int:
     if not matrix: return 0

     m = len(matrix)
     n = len(matrix[0])

     left = [0] * n # initialize left as the leftmost boundary possible
     right = [n] * n # initialize right as the rightmost boundary possible
     height = [0] * n

     maxarea = 0

     for i in range(m):

         cur_left, cur_right = 0, n
         # update height
         for j in range(n):
             if matrix[i][j] == '1': height[j] += 1
             else: height[j] = 0
         # update left
         for j in range(n):
             if matrix[i][j] == '1': left[j] = max(left[j], cur_left)
             else:
                 left[j] = 0
                 cur_left = j + 1
         # update right
         for j in range(n-1, -1, -1):
             if matrix[i][j] == '1': right[j] = min(right[j], cur_right)
             else:
                 right[j] = n
                 cur_right = j
         # update the area
         for j in range(n):
             maxarea = max(maxarea, height[j] * (right[j] - left[j]))

     return maxarea

```

