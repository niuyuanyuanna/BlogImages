---
title: leetcode3
date: 2018-05-23 16:54:21
tags:
- leetcode
- java
categories: Leetcode
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/leetcode.png
---

# 33. Search in Rotated Sorted Array 

## Description

Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.

(i.e., `[0,1,2,4,5,6,7]` might become `[4,5,6,7,0,1,2]`).

You are given a target value to search. If found in the array return its index, otherwise return `-1`.

You may assume no duplicate exists in the array.

Your algorithm's runtime complexity must be in the order of *O*(log *n*).

## Example

1. 

```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4
```

2. 

```
Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1

```



## Solution

这里使用二分法查找最小的数，然后再将数组分为两部分，这两部分都是顺序排列的，再使用二分法查找目标数是否在该数组中。

```java
class Solution {
    public static int search(int[] nums, int target) {
        if (nums.length == 0 || (nums.length == 1 && nums[0] != target)) return -1;
        if (nums.length == 1 && nums[0] == target) return 0;

        int left = 0;
        int right = nums.length - 1;
        int ans = -1;
        while(left < right){
            int mid = (left + right) / 2;
            if (nums[mid] > nums[right])
                left = mid + 1;
            else right = mid;
        }

        if (left > 0){
            if (target < nums[left] || target > nums[left-1]) return -1;
            else if (target >= nums[0] && target <= nums[left-1])
                ans = searchIndex(nums, target, 0, left-1);
            else if (target >= nums[left] && target <= nums[nums.length -1])
                ans = searchIndex(nums, target, left, nums.length -1);

        }else{
            if(target < nums[0] || target > nums[nums.length -1]) return -1;
            ans = searchIndex(nums, target, 0, nums.length -1);
        }
        return ans;
    }

    private static int searchIndex(int[] nums, int target, int left, int right){
        if (target == nums[left]) return left;
        if (target == nums[right]) return right;
        while(left <= right){
            int mid = (left + right) / 2;
            if (nums[mid] == target) return mid;
            else if(nums[mid] > target) right = mid-1;
            else left = mid + 1;
        }
        return -1;
    }

}
```

