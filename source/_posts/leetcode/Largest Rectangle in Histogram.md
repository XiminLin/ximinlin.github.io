---
title: Largest Rectangle in Histogram
categories:
  - leetcode
tags:
  - hard
  - stack
  - DP
---





# Largest Rectangle in Histogram #hard

Given an array of integers `heights` representing the histogram's bar height where the width of each bar is `1`, return _the area of the largest rectangle in the histogram_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/04/histogram.jpg)

**Input:** heights = [2,1,5,6,2,3]
**Output:** 10
**Explanation:** The above is a histogram where width of each bar is 1.
The largest rectangle is shown in the red area, which has an area = 10 units.

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/01/04/histogram-1.jpg)

**Input:** heights = [2,4]
**Output:** 4

**Constraints:**

-   `1 <= heights.length <= 105`
-   `0 <= heights[i] <= 104`

# Solution:

### Brute force

two pointers i, j; j keeps growing to the right. O(n^2)/O(1)



### ==divide and conquer==

O(nlogn)/==O(n)== **recursion depth O(n)**
最大的 rectangle 只能是：

1. 当前最长 width * shortest height

2. shortest height 的左边里面接着找

3. shortest height 右边接着找

  ![Divide and Conquer](https://leetcode.com/media/original_images/84_Largest_Rectangle2.PNG)

  

```c++
class Solution {
public:
	int calculateArea(vector<int>& heights, int start, int end) {
		if (start > end) {
			return 0;
		}
		int min_index = start;
		for (int i = start; i <= end; i++) {
			if (heights[min_index] > heights[i]) {
				min_index = i;
			}
		}
		return max({heights[min_index] * (end - start + 1),
					calculateArea(heights, start, min_index - 1),
					calculateArea(heights, min_index + 1, end)});
	}

	int largestRectangleArea(vector<int>& heights) {
		return calculateArea(heights, 0, heights.size() - 1);
	}
};

```

> better divide and conquer uses **Segment Tree**



==### stack method==

==**O(n)/O(n)**==

做 increasing heights stack, 因为 heights 在里面是 increasing 的，如果出现 新的 height 比 stack top 小，则 `heights[stack[j]..stack[k]] > new_height` ，
==***这些 heights 所能得到的 最大的 width 就已经确定了，可以计算这些 heights 能得到的 area 了***==

> 注意这里

```c++
class Solution {
public:
	int largestRectangleArea(vector<int>& heights) {
		stack<int> stk;
		stk.push(-1); //代表 end of stack
		int max_area = 0;
		for (size_t i = 0; i < heights.size(); i++) {
			while (stk.top() != -1 and heights[stk.top()] >= heights[i]) {
				int current_height = heights[stk.top()];
				stk.pop();
				int current_width = i - stk.top() - 1;
				max_area = max(max_area, current_height * current_width);
			}
			stk.push(i);
		}
		while (stk.top() != -1) {
			int current_height = heights[stk.top()];
			stk.pop();
			int current_width = heights.size() - stk.top() - 1;
			max_area = max(max_area, current_height * current_width);
		}
		return max_area;
	}
};
```

