---
title: 剑指offer刷题总结（数组）
date: 2018-07-12 00:00:08
tags:
- 剑指offer
- java
- python
- 数组
categories: Offer Problems
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/offer.jpg
---
# <center>剑指offer总结——数组</center>

### 面试3：数组中的重复数字
#### 题目描述
在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。
#### 解决方案
重排数组，指针从第一个下标开始，指针index等于数组中指针指向的数就移动到下一个，如果不是，就将指针指向数组中数字的index，直到指针index和它指向的数字相等。时间复杂度$O(n)$，空间复杂度$O(1)$。
- java解法
```java
public boolean duplicate(int numbers[],int length,int [] duplication) {
    if(numbers == null) return false;
    for(int i = 0; i < numbers.length; i++){
        if(numbers[i] < 0 || numbers[i] >= numbers.length) return false;
        while(numbers[i] != i){
            if(numbers[i] == numbers[numbers[i]]){
                duplication[0] = numbers[i];
                return true;
            }
            int temp = numbers[i];
            numbers[i] = numbers[temp];
            numbers[temp] = temp;
        }
    }
    return false;
}
```
- python解法

```python
def duplicate(self, numbers, duplication):
    if numbers == None:
        return False
    for i in range(len(numbers)-1):
        if numbers[i] < 0 or numbers[i] >len(numbers)-1:
            return False
        while numbers[i] != i:
            if numbers[i] == numbers[numbers[i]]:
                duplication[0] = numbers[i]
                return True
            temp = numbers[i]
            numbers[i] = numbers[temp]
            numbers[temp] = temp
    return False
```
### 面试4：二位数组中的查找
#### 题目描述
在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
#### 解决方案
选取左下角的数字作为起始点，如果目标数比这个数字大，就往右走一步，比目标数字小就往左走一步，若下标越界则没找到，返回false。时间复杂度为$O(n)$，空间复杂度为$O(1)$

- java解法
```java
public boolean Find(int target, int [][] array) {
    int n = array.length;
    int m = array [0].length;
    int i = n - 1, j = 0;
    while(i >= 0 && j < m){
        if (target == array[i][j]) return true;
        else if (target > array[i][j]) j++;
        else i--;
    }
    return false;
}
```

- python解法

```python
def Find(self, target, array):
    # write code here
    if not isinstance(target, (int, float)):
        return False
    if array == None:
        return False
    m = len(array)
    n = len(array[0])
    i = m-1
    j = 0
    while i >= 0 and j < n:
        if array[i][j] == target:
            return True
        elif array[i][j] < target:
            j += 1
        else:
            i -= 1
    return False
```
### 面试题21：调整数组顺序使奇数位于偶数前面 
#### 题目描述
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。
#### 解决方案
用两个指针维护数组，第一个指针在头，第二个指针在尾，如果第一个指向偶数，第二个指向奇数，则交换两个数字。时间复杂度为$O(n)$，空间复杂度为$O(1)$。
##### 方法一
书中没有保持数组位置的相对稳定性，在空间复杂度为$O(1)$的情况下需要使用插入排序或归并排序的方法。此时的时间复杂度为$O(n^{2})$。

- java解法
```java
public void reOrderArray(int [] array) {
    if(array == null || array.length == 1) return;
    for(int i = 1; i < array.length; i++){
        if((array[i] & 1) == 1){
            for(int j = i; j > 0; j--){
                if((array[j - 1] & 1) == 0){
                    int temp = array[j];
                    array[j] = array[j-1];
                    array[j-1] = temp;
                }
            }
        }
    }
}

```
##### 方法二

使用python直接写简单的排序数组，顺序不变，时间复杂度$O(n)$，空间复杂度$O(n)$。

- python解法
```python
def reOrderArray(self, array):
    if array is None or len(array) == 1:
        return array
    left = [x for x in array if x & 1]
    right = [x for x in array if not x & 1]
    return left + right
```

##### 方法三

可扩展解法，希望解决同一类的问题，而不仅是奇数和偶数的排序，可能是将数组分为负数和非负数，大于某数和小于等于某数。

- python解法

```python
def reOrderArray(self, array):
    if array is None or len(array) == 1:
        return array
    left = [x for x in array if x & 1]
    right = [x for x in array if not x & 1]
    return left + right
def Reorder(self, pData, length, func):
    if length == 0:
        return
    pBegin = 0
    pEnd = length - 1
    while pBegin < pEnd:
        while pBegin < pEnd and not func(pData[pBegin]):
            pBegin += 1
        while pBegin < pEnd and func(pData[pEnd]):
            pEnd -= 1
        if pBegin < pEnd:
            pData[pBegin], pData[pEnd] = pData[pEnd], pData[pBegin]
    return pData

def isEven(self, n):
    return not n & 0x1

def isNegtive(self, n):
    return n >= 0

def ReorderOddEven(self, pData):
    length = len(pData)
    return self.Reorder(pData, length, func=self.isNegtive)
```

### 面试题29：数组中出现次数超过一半的数字
#### 题目描述
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。
#### 解题思路
##### 方法一
使用快排的方法，判断选中的数字是否是数组的中位数。时间复杂度为$O(n)$

- java解法
```java
public int MoreThanHalfNum_Solution(int [] array) {
    if (array == null) return 0;
    int mid = array.length >> 1, l = 0, r = array.length - 1;
    int index = partition(array, l, r);
    while(index != mid){
        if(index > mid){
            index = partition(array, l, index - 1);
        }else{
            index = partition(array, index + 1, r);
        }
    }
    int result = array[mid];
    if(!checkValid(array, result)) return 0;
    return result;
}
public boolean checkValid(int[] array, int target){
    if(array == null) return false;
    int count = 0;
    for(int i = 0; i < array.length; i++){
        if(array[i] == target){
            count++;
        }
    }
    if(count > (array.length / 2)) return true;
    else return false;
}
public int partition(int[] array, int l, int r){
    int index = (int)(Math.random()*(r-l+1)) + l;
    swap(array, index, r);
    int small = l - 1;
    for(index = l ; index < r; index++){
        if(array[index] < array[r]){
            small++;
            if(array[small] != array[index]){
                swap(array, small, index);
            }
        }
    }
    small++;
    swap(array, small, r);
    return small;
}
public void swap(int[] array, int i, int j){
    int temp = array[i];
    array[i] = array[j];
    array[j] = temp;
}
```
##### 方法二
根据数组的特点找到重复次数最大的数字。使用两个数字存储，一个存储当前数字，一个存储time，当下一个数字和之前保存的数字相同时就time++，不同时time--，当次数为0的时候保存下一个数字并把当前次数设为1。

- python
```python
def MoreThanHalfNum_Solution(self, numbers):
    if numbers is None:
        return 0;
    temp = numbers[0]
    time = 1
    for i in range(1, len(numbers) - 1):
        if time == 0:
            temp = numbers[i]
            time = 1;
        if numbers[i] == temp:
            time += 1
        else:
            time -= 1
    if self.checkValid(numbers, temp):
        return temp
    else:
        return 0


def checkValid(self, numbers, target):
    count = 0
    for num in numbers:
        if num == target:
            count += 1
    if count > (len(numbers) / 2):
        return True
    else:
        return False
```

### 面试题42：连续子数组的最大和 
#### 题目描述
在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和。(子向量的长度至少是1)，要求时间复杂度为$O(n)$。
#### 解题思路
使用两个变量存储数据，第一个存储当前遍历的数组的和，当和<0时抛弃前面累加的数字。第二个存储当前遍历的最大子数组的和，当和值大于之前存储的最大和时更新最大和。空间复杂度为$O(n)$。

- java解法
```java
public int FindGreatestSumOfSubArray(int[] array) {
    if (array.length == 0) return 0;
    if (array.length == 1) return array[0];
    int ans = array[0];
    int max = ans;
    for (int i = 1; i<array.length; i++){
        max = Math.max(max + array[i], array[i]);
        ans = Math.max(ans, max);
    }
    return ans;
}
```
### 面试题45：把数组排成最小的数 
#### 题目描述
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。
#### 解题思路
将数组整型元素转换为字符串元素，采用字符串元素之间的比较规则判断，重写compare接口，将String数组排序，最后将排好序的字符串数组拼接出来。关键就是制定排序规则。

 - 先将整型数组转换成String数组，然后
 - 排序规则如下：
	- 若ab > ba 则 a > b，
	- 若ab < ba 则 a < b，
	- 若ab = ba 则 a = b；

 如： "3" < "31"但是 "331" > "313"，所以要将二者拼接起来进行比较。

 - java解法
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.StringBuilder;

public String PrintMinNumber(int[] numbers) {
    if (numbers.length == 0) return "";
    StringBuilder ans = new StringBuilder();
    ArrayList<Integer> list = new ArrayList<>();
    for (int number : numbers){
        list.add(number);
    }
    list.sort(new Comparator<Integer>(){
        @Override
        public int compare(Integer num1, Integer num2){
            String str1 = num1 + "" + num2;
            String str2 = num2 + "" + num1;
            return str1.compareTo(str2);
        }
    });
    for (int num : list){
        ans.append(num);
    }
    return ans.toString();
}
```
### 面试题49：丑数 
#### 题目描述
把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
#### 解题思路
一种传统的解法是判断所有整数是不是丑数，是则count++，不是就i++，直到count=N，这种解法时间复杂度过高，每个整数都需要计算。
另一种方法是使用空间换时间，创建数组保存已经找到的丑数。使用三个变量存储数组中最后一个×2后大于M的丑数，记做$M_{2}$，及×3后大于M的丑数，记做$M_{3}$，×5后大于M的丑数，记做$M_{5}$。每次生成新的丑数时更新$M_{2}$，$M_{3}$，$M_{5}$。用$T_{2}$、$T_{3}$、$T_{5}$，记录$M_{2}$，$M_{3}$，$M_{5}$的index。

这种方法使用$O(N)$的空间来换取时间，时间复杂度也为$O(N)$。

- java解法

```java
public int GetUglyNumber_Solution(int index) {
    if (index < 7) return index;
    ArrayList<Integer> list = new ArrayList<>();
    list.add(1);
    int T2 = 0, T3 = 0, T5 = 0;
    while(list.size() < index){
        list.add(Math.min(list.get(T2) * 2, list.get(T3) * 3, list.get(T5) * 5 ));
        if (list.get(list.size() - 1) == list.get(T2) * 2) T2++;
        if (list.get(list.size() - 1) == list.get(T3) * 3) T3++;
        if (list.get(list.size() - 1) == list.get(T5) * 5) T5++;
    }
    return list.get(list.size() - 1);   
}
```

### 面试题50：第一个只出现一次的字符 

#### 题目描述
在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1。
#### 解题思路
使用长度为256的HashMap来存储出现过的字符。key是char的ASCII码的值，value是出现的次数。总体的时间复杂度为$O(n)$，使用一个长度为256的HashMap辅助空间，相当于大小为1KB，空间复杂度为$O(1)$。

- java解法
```java
public int FirstNotRepeatingChar(String str) {
    if (str.length() == 0) return -1;
    char[] list = str.toCharArray();
    int[] map = new int[255];
    for (int i = 0; i < list.length; i++){
        map[list[i]]++;
    }
    for (int j = 0; j < list.length; j++){
        if (map[list[j]] == 1) return j;
    }
    return -1;
}
```
### 题目变形：字符流中第一个不重复的字符
#### 题目描述
请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。
输出描述：
如果当前字符流没有存在出现一次的字符，返回#字符。
#### 解题思路
字符只能一个接一个地从字符流中读出，定义数据容器保存字符在字符流中的位置。当一个字符没有出现过时，就将它在字符流中的位置保存到数据容器中，当再次从字符流中出现时，忽略这个字符，把这个字符在容器中对应的值变为负数。

- java解法
```java
public class Solution {
    //Insert one char from stringstream
    public int[] map = new int[255];
    public index = 1;
    public void Insert(char ch){
        if (map[ch] == 0){
            map[ch] = index++;
        }else{
            map[ch] = -1;
        }
    }
    //return the first appearence once char in current stringstream
    public char FirstAppearingOnce(){
        int min = Integer.MAX_VALUE;
        char ans = '#';
        for (int i = 0; i < 256; i++){
            if (map[i] > 0 && map[i] < min){
                min = map[i];
                ans = (char)i;
            }
        }
        return ans;
    }
}
```

### 面试题51：数组中的逆序对 
#### 题目描述
在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007
输入描述：
题目保证输入的数组中没有的相同的数字
数据范围：
	对于%50的数据,size<=10^4
	对于%75的数据,size<=10^5
	对于%100的数据,size<=2*10^5

#### 解题思路
使用分而治之的方法，先把数组分为两部分，统计出数组内部的逆序对数目，再统计出两个相邻子数组之间的逆序对数目。实际就是归并排序的思想。将时间复杂度降低为$O(nlogn)$。





