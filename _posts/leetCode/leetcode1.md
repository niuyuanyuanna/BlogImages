---
title: Leetcode 1
date: 2018/5/16 21:56
tags:
- leetcode
- java
categories: Leetcode
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/leetcode.png
---
# 2.Add Two Numbers 

## Description

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.



## Example

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```



## Solution

主要需要注意的地方：

- 使用temp节点进行节点的后移，返回的l3其实是temp的前一个节点
- 在创建新节点时直接将carray的值赋给新节点
- 当进行到最后一位时不再创建新节点

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
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {              
        ListNode l3 = new ListNode(0);
        ListNode temp = l3;
        int carray = 0;
        while((l1 != null) || (l2 != null)){
            if(l1 != null){
                temp.val += l1.val;
                l1 = l1.next;
            }
            if(l2 != null){
                temp.val += l2.val;
                l2 = l2.next;
            }
            carray = temp.val / 10;
            temp.val = temp.val % 10;
            
            if(carray >0){
                temp.next = new ListNode(carray);
                temp = temp.next;
            }else if((l1 != null) || (l2 != null)){
                temp.next = new ListNode(0);
                temp = temp.next;
            }                
        }
        return l3;          
    }
}
```

 ## Debug

调试时需要注意，该函数返回的值实际上相当于l3头节点，在主函数中的操作为：

```java
public static void main(String[] args) {
        ListNode l1 = new ListNode(5);
        l1.next = new ListNode(7);
        l1.next.next = new ListNode(2);

        ListNode l2 = new ListNode(8);
        l2.next = new ListNode(7);
        l2.next.next = new ListNode(3);

        ListNode l3 = addTwoNumbers(l1, l2);
    }
```

# 3.Longest Substring Without Repeating Characters

## Description
Given a string, find the length of the longest substring without repeating characters.

## Examples

- Given "`abcabcbb`", the answer is "`abc`", which the length is 3.
- Given "`bbbbb`", the answer is "`b`", with the length of 1.
- Given "`pwwkew`", the answer is "`wke`", with the length of 3. Note that the answer must be a substring, "`pwke`" is a subsequence and not a substring.

## Solution
要点：采用滑动窗来寻找窗中的相同字符。滑动窗通常用于解决字符串问题，主要是一个$[i, j)$的区间，j向右滑动，找到重复的元素时，更新i值。将i的值变为重复元素的后一位，缩小区间。区间的长度就是不重复最大子序列的长度，为$j-i+1$。
注意：

- 使用HashMap存储字符及索引
- 遇到重复元素时，可能碰到重复元素索引值+1比原来的i值小的情况，此时不更新i
- 子序列的长度取最大的那个值
算法的时间复杂度为$O(n)$。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer> charMap = new HashMap<>();
        int n = s.length();
        int subLenth = 0, i = 0, j = 0;
        while(i < n && j < n){
            if (charMap.containsKey(s.charAt(j))){
                i = Math.max(charMap.get(s.charAt(j)) + 1, i);
            }
            charMap.put(s.charAt(j), j);
            subLenth = Math.max(j-i+1, subLenth);
            j++;
        }
        return subLenth;
    }
}
```
# 5.Longest Palindromic Substring

## Description
Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

## Example

1. 
```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```
2. 
```
Input: "cbbd"
Output: "bb"
```

## Solution
1. Manacher算法
```java
    public static String longestPalindrome(String s) {
        StringBuilder extS = new StringBuilder("*#");
        for (int i = 0; i < s.length(); i++){
            extS.append(s.charAt(i)).append("#");
        }
        int m = extS.length();
        int [] rs = new int[m];
        int maxEdge = 0, center = 0, start = 0, end = 0;
        for (int j= 1; j < m; j++){
            rs[j] = (maxEdge > j ? Math.min(rs[2 * center - j], maxEdge - j) : 1);

            while(extS.charAt(j + rs[j]) == extS.charAt(j - rs[j])){
                rs[j]++;
            }

            if (j+rs[j] > maxEdge){
                center = j;
                maxEdge = j + rs[j];
            }

            if (rs[j] - 1 > end){
                start = (j - rs[j]) / 2;
                end = rs[j] - 1;
            }
        }
        return s.substring(start, end);
    }

```

2. 动态规划
动态规划解法较为经典，采用一个转移矩阵`p[i][j]`来存储`s[i]` ~`s[j]`的子字符串是否为回形字符串。当`s[i]` ~`s[j]`为回形字符串时，其判断依据为：

- `s[i]`必然等于`s[j]`
- `s[i]` ~`s[j]`的上一组`s[i+1]` ~`s[j-1]`仍然是回形字符串。或者是碰到“bb”这种情况或“bab”，此时的j - i < 3。
采用动态规划仍然会有$O(n^{2})$的时间复杂度。

- 算法从后向前遍历，防止在边界条件时，`s[i+1]` ~`s[j-1]`溢出。

```java
    public static String longestPalindrome(String s) {
        int n = s.length();
        String ans = null;
        boolean [][] dp = new boolean[n][n];
        for (int i = n-1; i >= 0 ; i--) {
            for (int j = i; j < n; j++){
                if (s.charAt(i) == s.charAt(j) && (j - i < 3 || dp[i + 1][j - 1])){
                    dp[i][j] = true;
                }else {
                    dp[i][j] = false;
                }

                if (dp[i][j] && (ans == null || j - i + 1 > ans.length())){
                    ans = s.substring(i, j+1);
                }
            }

        }
        return ans;
```

# 6. ZigZag Conversion

## Description
The string` "PAYPALISHIRING" `is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)
```
P   A   H   N
A P L S I I G
Y   I   R
```
And then read line by line:` "PAHNAPLSIIGYIR"`

Write the code that will take a string and make this conversion given a number of rows:
```
string convert(string s, int numRows);
```

## Example:
1. 
```
Input: s = "PAYPALISHIRING", numRows = 3
Output: "PAHNAPLSIIGYIR"
```
2.
```
Input: s = "PAYPALISHIRING", numRows = 4
Output: "PINALSIGYAHRPI"
Explanation:

P     I    N
A   L S  I G
Y A   H R
P     I
```
 ## Solution
 1. 较为复杂的解法，首先需要将原字符串补全为可以被周期整除的字符串，并在后面补上`'*'`。然后可以观察到一个周期内访问字符串的顺序，根据这个顺序可以发现除了头部和尾部的字符，其他地方的字符在一个周期内是关于底部字符对称访问的。这里为了省去边界判断，直接在底部字符和周期尾部字符处插入`'*'`，使得在一个周期内字符的访问都是一个是周期头部，一个是周期末尾字符。

 ```java
    public String convert(String s, int numRows) {
        if (numRows == 1){
            return s;
        }
        StringBuffer ans = new StringBuffer();
        StringBuffer strBuff = new StringBuffer(s);
        StringBuffer strBuff2 = new StringBuffer();
        int n = s.length(), oneSize = numRows * 2 - 2;
        int m = n / oneSize;

        if (n % oneSize > 0){
            for (int i = 0; i < oneSize - n % oneSize; i++) {
                strBuff.append("*");
            }
            m++;
        }

        for (int i = 0; i < m; i++) {
            strBuff2.append(strBuff.substring(i*oneSize, numRows + i * oneSize))
                    .append("*")
                    .append(strBuff.substring(numRows + i * oneSize, oneSize + i * oneSize))
                    .append("*");
        }

        for (int i = 0; i < numRows; i++) {
            for (int j = 0; j < m; j++){
                if (strBuff2.charAt(i + j * (oneSize + 2)) != '*'){
                    ans.append(strBuff2.charAt(i + j * (oneSize + 2)));
                }
                if (strBuff2.charAt(oneSize - i + 1 + j * (oneSize + 2)) != '*'){
                    ans.append(strBuff2.charAt(oneSize - i + 1 + j * (oneSize + 2)));
                }
            }
        }
        return ans.toString();
    }
 ```

2. 分好周期后分情况讨论。
```java
    public String convert(String s, int numRows) {
        if(numRows == 0 || s == null || s == "") return "";
        if(numRows == 1) return s;

        StringBuilder sb = new StringBuilder();
        int size = 2 * numRows - 2;
        for(int i = 0; i < numRows; i++) {
            for(int j = i; j < s.length(); j += size) {
                sb.append(s.charAt(j));
                if(i != 0 && i != numRows - 1) {
                    int temp = j+ size - i * 2;
                    if(temp < s.length()) {
                        sb.append(s.charAt(temp));
                    }
                }
            }
        }
        return sb.toString();
    }
```

# 12. Integer to Roman

## Description

Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`. 

```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

For example, two is written as `II` in Roman numeral, just two one's added together. Twelve is written as, `XII`, which is simply `X` + `II`. The number twenty seven is written as `XXVII`, which is `XX` + `V` + `II`. 

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not `IIII`. Instead, the number four is written as `IV`. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as `IX`. There are six instances where subtraction is used: 

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9. 
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90. 
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given an integer, convert it to a roman numeral. Input is guaranteed to be within the range from 1 to 3999. 

## Example

1. 

```
Input: 3
Output: "III"
```

1. 

```
Input: 4
Output: "IV"
```

1. 

```
Input: 9
Output: "IX"
```

1. 

```
Input: 58
Output: "LVIII"
Explanation: C = 100, L = 50, XXX = 30 and III = 3.
```

1. 

```
Input: 1994
Output: "MCMXCIV"
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

## Solution



```
    public static String intToRoman(int num){
        String M[] = {"", "M", "MM", "MMM"};
        String C[] = {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"};
        String X[] = {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"};
        String I[] = {"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"};
        return M[num/1000] + C[(num%1000)/100] + X[(num%100)/10] + I[num%10];
    }
```







# 13. Roman to Integer 

## Description

Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`. 

```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

For example, two is written as `II` in Roman numeral, just two one's added together. Twelve is written as, `XII`, which is simply `X` + `II`. The number twenty seven is written as `XXVII`, which is `XX` + `V` + `II`. 

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not `IIII`. Instead, the number four is written as `IV`. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as `IX`. There are six instances where subtraction is used: 

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9. 
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90. 
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999. 

## Example

1. 

```
Input: "III"
Output: 3
```

2. 

```
Input: "IV"
Output: 4
```

3. 

```
Input: "IX"
Output: 9
```

4. 

```
Input: "LVIII"
Output: 58
Explanation: C = 100, L = 50, XXX = 30 and III = 3.
```

5. 

```
Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

## Solution

将罗马数字转换为整形数字，具体思路为：动态规划

- 将每一个字母对应为整型数字
- 动态规划，从后向前判断当前数字和其后面的数字的大小

```java
class Solution {
    public int romanToInt(String s){
        int temp = charToInt(s.charAt(s.length()-1));
        int ans = temp;
        for (int i = s.length()-2; i >= 0; i--) {
            if (charToInt(s.charAt(i)) < temp){
                ans = ans - charToInt(s.charAt(i));
            }else {
                ans += charToInt(s.charAt(i));
            }
            temp = charToInt(s.charAt(i));
        }
        return ans;
    }

    public int charToInt(char c){
        int num = 0;
        switch (c){
            case 'I':
                num = 1;
                break;
            case 'V':
                num = 5;
                break;
            case 'X':
                num = 10;
                break;
            case 'L':
                num = 50;
                break;
            case 'C':
                num = 100;
                break;
            case 'D':
                num = 500;
                break;
            case 'M':
                num = 1000;
                break;
            default:
                break;
        }
        return num;
    }
}
```

