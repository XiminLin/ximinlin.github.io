---
title: Trapping Rain Water
categories:
  - leetcode
tags:
  - hard
  - stack
  - DP
  - pointers
---



# Trapping Rain Water #hard

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

**Example 1:**

![img](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)

```
Input: height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
Explanation: The above elevation map (black section) is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.
```

**Example 2:**

```
Input: height = [4,2,0,3,2,5]
Output: 9
```


**Constraints:**

- `n == height.length`
- `1 <= n <= 2 * 104`
- `0 <= height[i] <= 105`


# Solution:

### Brute-force

==先考虑 brute-force, 对于每一个点，我们考虑左边的最高的墙和右边的最高的墙，最终能hold的水为 min(left_height, right_height) - curr_height. 这样就 O(n^2)/O(1)==



### DP

Brute-force 基础上我们用 **DP** 存下左边到右边的高度，和右边到左边的高度，之后直接求. O(n)/O(n)



### Stack

用一个 stack 来做，观察到如果有 **valley (本来都是 decreasing heights, 看到比 stk 顶部更大的 height**)，我们会去iterate直到比当前 height 高的时候，这个时候我们更新得到的water，同时把这个valley弄平掉。O(n)/O(n), faster than 2., in one pass.

>  我们往 **stack 中添加 index**，如果比当前 index 的 height 低，则继续添加；如果比 top() 的 height 大，证明出现了valley，pop 出小的部分，计算得到的水量计入总量；

```c++
int trap(vector<int>& height)
{
   int ans = 0, current = 0;
   stack<int> st;
   while (current < height.size()) {
	   while (!st.empty() && height[current] > height[st.top()]) {
		   int top = st.top();
		   st.pop();
		   if (st.empty())
			   break;
		   int distance = current - st.top() - 1;
		   int bounded_height = min(height[current], height[st.top()]) - height[top];
		   ans += distance * bounded_height;
	   }
	   st.push(current++);
   }
   return ans;
}

```



### ==**two pointers approach: O(n)/O(1)**==

Best explained here:

<https://youtu.be/XqTBrQYYUcc>

总结就是：

>  我们对于每个 index 都在求函数 ==g(i) = min(left_max(i), right_max(i) )==。这个类似于求 lower envelop, ==**遇到类似lower envelop问题就是要想到 two pointers 或者 binary search**==，这里介绍 two pointers 办法

>  <u>从左到右的顺序下</u>，利用 **left_max** 函数 non-decreasing 和 **right_max** 函数 non-increasing 的性质，每次对比决定 pointer i 还是 pointer j 往中间移动。

```java
class Solution {
    public int trap(int[] height) {
        // time : O(n)
        // space : O(1)
        if (height.length==0) return 0; 
        int left = 0, right = height.length-1; 
        int leftMax=0, rightMax=0; 
        int ans = 0; 
        while (left < right) {
            if (height[left] > leftMax) leftMax = height[left]; 
            if (height[right] > rightMax) rightMax = height[right];
          	// 因为 rightMax 越靠左越大，所以这里一定能 trap 住了
          	// 这左边的 water 肯定能被 trap 了
            if (leftMax < rightMax) {
                ans += Math.max(0, leftMax-height[left]); 
                left++; 
            } else {
                ans += Math.max(0, rightMax-height[right]); 
                right--; 
            }
        }
        return ans; 
    }
}
```





