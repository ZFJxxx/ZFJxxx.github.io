---
layout: post
title:  Leetcode 删除或者移动数组中的元素
date:   2018-04-22 11:15:10
categories: 算法
tags: LeetCode
keywords: LeetCode
description: 
---

## LeetCode 27. Remove Element

Given an array and a value, remove all instances of that value in-place and return the new length.
 
Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

The order of elements can be changed. It doesn’t matter what you leave beyond the new length.
```
Example:

Given nums = [3,2,2,3], val = 3,

Your function should return length = 2, with the first two elements of nums being 2.
```
### 解

一个数组，要除掉所有值为val的元素之后，返回新数组的长度。

和Move Zeros很像，解题方式相同。也是用两个指针，指针i遍历数组，指针j指向值为val的元素。当nums[i] 不为val时，将nums[i]的值赋给nums[j]

只要将所有合格的元素移动到前端，算出数组长度就行了
```
class Solution {
    public int removeElement(int[] nums, int val) {
        int j = 0;
        for(int i = 0 ; i<nums.length;i++)
            if(nums[i] != val) 
                nums[j++] = nums[i];
        return j;
    }
}
```

## Leetcode 283. Move Zeros

Given an array nums, write a function to move all 0’s to the end of it while maintaining the relative order of the non-zero elements.

For example, given nums = [0, 1, 0, 3, 12], after calling your function, nums should be [1, 3, 12, 0, 0].

Note: 
You must do this in-place without making a copy of the array. 
Minimize the total number of operations.

### 解
要求很简单，把数组中所有的0元素移到数组最后面去。 
我的思路是，两个指针i,j指向头部，指针i遍历，每当遇到不为零的数就赋值给Nums[j]，最后将指针j后面全部赋予0值就行了。

只要将所有合格的元素移动到最前端就行了，后面补0 ，时间复杂度o(n)
```
class Solution {
    public void moveZeroes(int[] nums) {
        if (nums == null || nums.length == 0) return; 
        int j = 0;
        for(int i = 0; i<nums.length;i++){
            if(nums[i] != 0)  nums[j++] = nums[i];
        }

        while(j < nums.length){
            nums[j++] = 0;
        }   
    }
}
```

## LeetCode 26.Remove Duplicates from Sorted Array

Given a sorted array, remove the duplicates in-place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

Example:

Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of 
nums being 1 and 2 respectively. It doesn’t matter what you leave 
beyond the new length.

### 解

删除掉有序数组内所有重复的元素，只保留一个。

思路是两个指针i，j，指针i遍历数组，每当nums[i]和nums[j]不同时，j就往前移动一格然后num[i]的值赋予给nums[j]
```
class Solution {
    public int removeDuplicates(int[] nums) {
        int j =0;
        for(int i=0; i<nums.length;i++){
            if(nums[i] != nums[j]) 
                nums[++j]= nums[i];
        }
        return j+1;
    }
}
```

## LeetCode 80. Remove Duplicates from Sorted Array II

Follow up for “Remove Duplicates”: 
What if duplicates are allowed at most twice?

For example, 
Given sorted array nums = [1,1,1,2,2,3],

Your function should return length = 5, with the first five elements of nums being 1, 1, 2, 2 and 3. It doesn’t matter what you leave beyond the new length.

### 解
有序数组去重，每个重复元素最多保留两个。 和LeetCode 26.Remove Duplicates from Sorted Array 相似，只不过这里每个重复的元素可以保留两个。
```
class Solution {
    public int removeDuplicates(int[] nums) {
        int i,j;
         for(i=2,j=2 ; i<nums.length;i++)
             if(nums[j-2]!=nums[i]) 
                 nums[j++]=nums[i];
         return j;
    }
}
```
这类问题有个总结的解法 如果是可以保留重复k个元素
```
class Solution {
    public int removeDuplicates(int[] nums，int k) {
        if(nums.size()<k) return nums.size(); // if array size is less than k then return the same
        int i,j;
         for(i=k,j=k ; i<nums.length;i++)
             if(nums[j-k]!=nums[i]) nums[j++]=nums[i];
         return j;
    }
 }
 ```
