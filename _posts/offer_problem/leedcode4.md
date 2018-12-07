---
title: leedcode4
date: 2018-11-08 16:36:18
tags:
- leetcode
- python
categories: Leetcode
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/LeetCode.png
---

# 114. Flatten Binary Tree to Linked List

## Description

Given a binary tree, flatten it to a linked list in-place.
For example, given the following tree:
    1
   / \
  2   5
 / \   \
3   4   6
The flattened tree should look like:

1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6

## Hints
If you notice carefully in the flattened tree, each node's right child points to the next node of a pre-order traversal.

## Method

非递归的先序遍历，使用栈临时存储节点，先存右子节点，再存左子节点



# 4. Median of Two Sorted Arrays
There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be$ O(log (m+n))$.

You may assume nums1 and nums2 cannot be both empty.

## Method
cut A and B into two parts:

|        left_part        |        right_part         |
| :---------------------: | :-----------------------: |
| A[0], A[1], ..., A[i-1] | A[i], A[i+1], ..., A[m-1] |
| B[0], B[1], ..., B[j-1] | B[j], B[j+1], ..., B[n-1] |

there is two conditions we should make sure:

- `len(left_part) == len(right_part)`
- `max(left_part) <= min(right_part)`

so, for the aboving condition, the equivalent should be：
$$
i + j = m - i + n - j \quad and \quad m \leq n\\
B[j -1] \leq A[i] \quad and \quad A[i - 1] \leq B[j]
$$
the condition $m \leq n$ means $j \geq 0$。

then the median of two arrays is:

```
median = (max(left_part) + min(right_part)) / 2    // when m + n is even
median = max(A[i-1], B[j-1])                       // when m + n is odd
```

the mission can be solved by the following steps:

- find min(len(array1), len(array2)) and set it to m
- searching i in [0, m] to find the 'i' that:
  - `B[j - 1] <= A[i]` and `A[i -1] <= B[j]` where `j =  (m + n + 1)/2 - i`

if the time complexity is $ O(log (min(m+n)))$,we should use binary search to find the 'i'

## Pseudo-code

```
<1> Set imin = 0, imax = m, then start searching in [imin, imax]
<2> Set i = (imin + imax)/2, j = (m + n + 1)/2 - i
<3> Now we have len(left_part)==len(right_part). And there are only 3 situations
     that we may encounter:
     <a> B[j-1] <= A[i] and A[i-1] <= B[j]
         means we have found the object 'i', stop searching
     <b> B[j-1] > A[i]
         means A[i] is too small, 'i' should be increased, so set:
         imin = i + 1;
         go to <2>
     <c> A[i-1] > B[j]
         means A[i-1] is too big, 'i' should be decreased, so set:
         imax = i - 1;
         go to <2>
     
```

when it comes to eadge values  **i=0,i=m,j=0,j=n**, where** A[i-1],B[j-1],A[i],B[j]** may not exist.



