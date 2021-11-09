---
title: First Missing Positive
categories:
  - leetcode
tags:
  - hard
  - array
  - value2idx
---



# First Missing Positive #hard

Given an unsorted integer array `nums`, return the smallest missing positive integer.

You must implement an algorithm that runs in `O(n)` time and uses constant extra space.

**Example 1:**

**Input:** nums = [1,2,0]
**Output:** 3

**Example 2:**

**Input:** nums = [3,4,-1,1]
**Output:** 2

**Example 3:**

**Input:** nums = [7,8,9,11,12]
**Output:** 1

**Constraints:**

-   `1 <= nums.length <= 5 * 105`
-   `-231 <= nums[i] <= 231 - 1`


# Solution:

分析问题：对于长度为 n 的 array，最大的 missing positive 可能是几

![max_first](https://leetcode.com/problems/first-missing-positive/Figures/41/41_max_possible_first.png)

这个example我们发现是 9，最大为 n+1. 最小为 1.

这种就考虑用 value hash to index 的方法来做 #value_hash_to_index 

1. 对于所有的 <=0 和 >= n+1 的数都设置为 0
2. 遍历 array，标记 ```nums[ nums[i] ] = -1 * abs(nums[nums[i]])*```，使其为负来标记存在
3. 最后遍历，寻找 index 里是 非负数的 格子，得到答案

```python
class Solution:
    def firstMissingPositive(self, nums: List[int]) -> int:
        n = len(nums)
        
        # Base case.
        if 1 not in nums:
            return 1
        
        # Replace negative numbers, zeros,
        # and numbers larger than n by 1s.
        # After this convertion nums will contain 
        # only positive numbers.
        for i in range(n):
            if nums[i] <= 0 or nums[i] > n:
                nums[i] = 1
        
        # Use index as a hash key and number sign as a presence detector.
        # For example, if nums[1] is negative that means that number `1`
        # is present in the array. 
        # If nums[2] is positive - number 2 is missing.
        for i in range(n): 
            a = abs(nums[i])
            # If you meet number a in the array - change the sign of a-th element.
            # Be careful with duplicates : do it only once.
            if a == n:
                nums[0] = - abs(nums[0])
            else:
                nums[a] = - abs(nums[a])
            
        # Now the index of the first positive number 
        # is equal to first missing positive.
        for i in range(1, n):
            if nums[i] > 0:
                return i
        
        if nums[0] > 0:
            return n
            
        return n + 1
```

