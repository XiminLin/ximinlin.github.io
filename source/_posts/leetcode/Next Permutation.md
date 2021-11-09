---
title: Next Permutation
categories:
  - leetcode
tags:
  - medium
  - array
---



# Next Permutation

Implement **next permutation**, which rearranges numbers into the lexicographically next greater permutation of numbers.

If such an arrangement is not possible, it must rearrange it as the lowest possible order (i.e., sorted in ascending order).

The replacement must be **[in place](http://en.wikipedia.org/wiki/In-place_algorithm)** and use only constant extra memory.

 

**Example 1:**

```
Input: nums = [1,2,3]
Output: [1,3,2]
```

**Example 2:**

```
Input: nums = [3,2,1]
Output: [1,2,3]
```

**Example 3:**

```
Input: nums = [1,1,5]
Output: [1,5,1]
```

**Example 4:**

```
Input: nums = [1]
Output: [1]
```

 

**Constraints:**

- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 100`



# Solution

**==Make up some examples of numbers==**, and the algorithm for finding the next permutations is:

1. Iterate from right to left. If the number is increasing, the *sub-number* we have seen is in its largest. 
2. Find the first index $i$ where the in adjaceny,  **left number num[i-1] is greater than the right number num[i]**. Swap these two numbers. We are sure that `num[0]..num[i],num[i-1],...` **must share the first $i$ values with the smallest permutation** 不可能更小了
3. Then, we can make sure `num[i+1],...num[n]` are the smallest by reversing them. (因为我们刚才从右到左都是 increasing，现在可以让他们从左到右都是increasing即可)



```java
public class Solution {
    public void nextPermutation(int[] nums) {
        int i = nums.length - 2;
        while (i >= 0 && nums[i + 1] <= nums[i]) {
            i--;
        }
        if (i >= 0) {
            int j = nums.length - 1;
            while (j >= 0 && nums[j] <= nums[i]) {
                j--;
            }
            swap(nums, i, j);
        }
        reverse(nums, i + 1);
    }

    private void reverse(int[] nums, int start) {
        int i = start, j = nums.length - 1;
        while (i < j) {
            swap(nums, i, j);
            i++;
            j--;
        }
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```





