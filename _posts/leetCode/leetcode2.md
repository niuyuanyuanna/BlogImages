---
title: leetcode2
date: 2018-05-22 14:56:37
tags:
- leetcode
- java
categories: Leetcode
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/leetcode.jpg
---

#  19. Remove Nth Node From End of List 

## Description

Given a linked list, remove the *n*-th node from the end of list and return its head. 



## Example

```
Given linked list: 1->2->3->4->5, and n = 2.

After removing the second node from the end, the linked list becomes 1->2->3->5.
```

**Note:**

Given *n* will always be valid.

## Solution

这里需要查找链表的最后第n个节点，并将其删除。主要思想是利用两个指针，第一个指针先走n步，第二个指针再开始走，当第一个指针移动到链表末端时，第二个指针正好指向需要移除的节点。

注意：

- 在头部添加一个节点，防止边界溢出。
- 在第一个指针移动到尾部时，第二个指针需要指向移除节点的前一个节点。
- 找到该节点后，将待移除节点的后一节点连接到该节点就相当于完成了节点的移除

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        if (n == 0) return head;
        ListNode deleteNode = new ListNode(0);
        deleteNode.next = head;
        ListNode first = deleteNode;
        ListNode second = deleteNode;
        int time = 0;
        while (first.next != null){
            first = first.next;
            time++;
            if(time > n){
                second = second.next;
            }
            
        }
        second.next = second.next.next;
        return deleteNode.next;          
    }
}
```

此算法的时间复杂度为$O(n)$，其中$n$为链表的长度。



# 20. Valid Parentheses 

## Description

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.



## Examp

1. 

```
Input: "()"
Output: true
```

2. 

```
Input: "()[]{}"
Output: true
```

3. 

```
Input: "(]"
Output: false
```

4. 

```
Input: "([)]"
Output: false
```

5. 

```
Input: "{[]}"
Output: true
```

 ## Solution

这一题可以运用到栈的知识，关于栈的应用中的递归嵌套。遇到左括号就入栈，遇到又括号就出栈。这样算法在最坏情况下的时间复杂度为$O(n)$，其中n为输入字符串的长度。这里使用一个StringBuilder来替代栈结构。采用栈Stack来进行相同的操作时，StringBuilder的处理时间会更短。

```java
    public boolean isValid(String s) {
        if (s.equals("")) return true;
        StringBuilder stringBuilder = new StringBuilder();
        int n = s.length();
        if (n % 2 > 0) return false;
        for (int i = 0; i <n; i++){
            char nowChar = s.charAt(i);
            if (nowChar == '(' || nowChar == '[' || nowChar == '{'){
                stringBuilder.append(s.charAt(i));
            }else if (nowChar == ')' || nowChar == ']' || nowChar == '}'){
                if (i == 0) return false;
                int m = nowChar - stringBuilder.charAt(stringBuilder.length() - 1);
                if (m < 3 && m > 0){
                    stringBuilder.deleteCharAt(stringBuilder.length() - 1);
                }else return false;
            }
        }
        if (stringBuilder.length() == 0) return true;
        else return false;
    }
```

# 21. Merge Two Sorted Lists 

## Description

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists. 

## Example

```
Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```

## Solution

按照常规解法，l1和l2逐次向下比较即可，时间复杂度为$O(n1+n2)$n1是l1的长度，n2是l2的长度。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public  static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;
        ListNode ans = new ListNode(0);
        ListNode ansCopy = ans;
        int temp = 0;
        while (l1 != null || l2 != null){
            if (l1 != null  && l2 != null){
                if (l1.val < l2.val){
                    temp = l1.val;
                    l1 = l1.next;
                }else {
                    temp = l2.val;
                    l2 = l2.next;
                }
            }else if (l1 != null && l2 == null){
                temp = l1.val;
                l1 = l1.next;
            }else if (l1 == null && l2 != null){
                temp = l2.val;
                l2 = l2.next;
            }
            ansCopy.next = new ListNode(temp);
            ansCopy = ansCopy.next;
        }
        return ans.next;
    }
}
```



# 22. Generate Parentheses

## Description

Given *n* pairs of parentheses, write a function to generate all combinations of well-formed parentheses. 

## Example

 given *n* = 3, a solution set is: 

```
[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

## Solution

采用递归的方法求解。每次递归，输出字符串增加一个括号，使用一个字符数组记录输出字符串。将"("剩余的数量记为`left`，“)”剩余的数量记为`right`。当`left > 0`时，递归调用函数，并将`left - 1`，输出字符数组对应位置存储“(”。右括号同理，不过右括号的剩余数量始终是大于左括号数目的，满足这个条件可以执行右括号的递归。最终字符数组存储完成后将结果添加到返回值中。

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> res = new ArrayList<>();
        if(n == 0) {
            return res;
        }
        char[] array = new char[2 * n];
        helper(array, 0, n, n, res);
        return res;
    }
    private void helper(char[] array, int index, int left, int right, List<String> res) {
        if(index == array.length) {
            res.add(new String(array));
            return;
        }
        if(left > 0) {
            array[index] = '(';
            helper(array, index + 1, left - 1, right, res);
        }
        if(right > left) {
            array[index] = ')';
            helper(array, index + 1, left, right - 1, res);
        }
    }
}
```



# 26. Remove Duplicates from Sorted Array 

## Description

Given a sorted array *nums*, remove the duplicates [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm) such that each element appear only *once* and return the new length.

Do not allocate extra space for another array, you must do this by **modifying the input array in-place** with O(1) extra memory.

## Example

1. 

```
Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.

It doesn't matter what you leave beyond the returned length.
```

2. 

```
Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.

It doesn't matter what values are set beyond the returned length.
```

## Clarification

Confused why the returned value is an integer but your answer is an array?

Note that the input array is passed in by **reference**, which means modification to the input array will be known to the caller as well.

Internally you can think of this:

```java
// nums is passed in by reference. (i.e., without making a copy)
int len = removeDuplicates(nums);

// any modification to nums in your function would be known by the caller.
// using the length returned by your function, it prints the first len elements.
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

## Solution

这题除了需要找到不重复的数组的长度之外，还需要更新数组，将刚找到的不相同的数置于`nums[ans - 1]`的位置。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;
        if (nums.length == 1) return 1;
        int ans = 1;
        int preNum = nums[0];
        for(int i = 1; i < nums.length; i++){
            if (nums[i] > preNum) {
                ans++;
                nums[ans - 1] = nums[i];
            }
            preNum = nums[i];
        }
        return ans;
    }
}
```



# 28.  Implement strStr() 

## Description

Implement [strStr()](http://www.cplusplus.com/reference/cstring/strstr/).

Return the index of the first occurrence of needle in haystack, or **-1** if needle is not part of haystack.

## Example

1. 

```
Input: haystack = "hello", needle = "ll"
Output: 2
```

2. 

```
Input: haystack = "aaaaa", needle = "bba"
Output: -1
```

## Clarification

What should we return when `needle` is an empty string? This is a great question to ask during an interview.

For the purpose of this problem, we will return 0 when `needle` is an empty string. This is consistent to C's [strstr()](http://www.cplusplus.com/reference/cstring/strstr/) and Java's [indexOf()](https://docs.oracle.com/javase/7/docs/api/java/lang/String.html#indexOf(java.lang.String)).



## Solution

本题首先用到一个技巧，先判断第二个字符串是否包含于第一个字符串，缩减判断时间。其次使用滑动窗，首尾节点的长度为第二个字符串的长度，沿着第一个字符串滑动，判断当前子字符串是否和第二个字符串相等。

```java
class Solution {
    public int strStr(String haystack, String needle) {
        if (needle.isEmpty()) return 0;
        if (!haystack.contains(needle)) return -1;
        for (int l = 0, r = needle.length()-1; l < haystack.length(); l++, r++){
            if (haystack.substring(l, r+1).equals(needle)) return l;
        }
        return -1;
    }
    
}
```

# 29. Divide Two Integers 

## Description

Given two integers `dividend` and `divisor`, divide two integers without using multiplication, division and mod operator.

Return the quotient after dividing `dividend` by `divisor`.

The integer division should truncate toward zero.

## Example

1. 

```
Input: dividend = 10, divisor = 3
Output: 3
```

2. 

```
Input: dividend = 7, divisor = -3
Output: -2
```

## Clarification

- Both dividend and divisor will be 32-bit signed integers.
- The divisor will never be 0.
- Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: $[−2^{31},  2^{31} − 1]$. For the purpose of this problem, assume that your function returns 231 − 1 when the division result overflows.

## Solution

此题困难之处在于需要提前对出书和被除数做出判断，如果其超出了整型数据的最大或最小值，则需要首先根据其符号返回整型的最大值或最小值。

具体步骤：

- 获取最终结果的符号
- 将除数和被除数转换为长整型计算。
- 分别判断被除数为0和除数为1的情况下的返回值。
- 递归实现其他正常情况下的结果。

```java
class Solution {
    public int divide(int dividend, int divisor) {
        if (divisor == 0)
            return Integer.MAX_VALUE;
        int symbol = 1;
        if ((dividend > 0 && divisor < 0) || (dividend < 0 && divisor > 0)) symbol = -1;

        long ldivident =  Math.abs((long)dividend);
        long ldivisor = Math.abs((long)divisor);

        if (ldivident == 0 || ldivident < ldivisor) return 0;
        if (ldivisor == 1){
            if (dividend == Integer.MIN_VALUE || dividend == Integer.MAX_VALUE){
                if (symbol == -1) return Integer.MIN_VALUE;
                else return Integer.MAX_VALUE;
            }else{
                return (int)(symbol * ldivisor);
            }
        }

        long lans = computeRes(ldivident, ldivisor);
        return (int)(lans * symbol);
    }

    private long computeRes(long ldividend, long ldivisor) {
        if (ldividend - ldivisor < 0) return 0;
        long sum = ldivisor;
        long temp = 1;
        while (sum + sum <= ldividend){
            sum += sum;
            temp += temp;
        }
        return temp + computeRes(ldividend-sum, ldivisor);
    }

}
```

