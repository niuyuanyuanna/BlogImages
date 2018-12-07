---
title: 常见的排序算法
date: 2018-07-19 12:23:37
tags:
- 剑指offer
- java
- python
- 排序
categories: Offer Problems
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/offer.jpg
---


# <center>常见的排序算法</center>

**冒泡排序、插入排序、选择排序、希尔排序、堆排序、归并排序、快速排序**

## 冒泡排序：

**平均时间复杂度$O(n^2)$，最好情况复杂度$O(n)$**

### 步骤：

1.  比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2.  对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3.  针对所有的元素重复以上的步骤，除了最后一个。
4.  持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

- java例子
```java
public void BubbleSort(int[] array){
    if (array.length <= 0) return;
    for (int i = 0; i < array.length; i++){
        for (int j = 0; j < array.length - i; j++){
            if (array[j] > array[j+1]){
                swap(arary, i, j);
            }
        }
    }
}
```


## 插入排序
**时间复杂度：$O(n^2)$，最优时间复杂度：$O(n)$**

### 步骤：

1.  从第一个元素开始，该元素可以认为已经被排序
2.  取出下一个元素，在已经排序的元素序列中从后向前扫描
3.  如果该元素（已排序）大于新元素，将该元素移到下一位置
4.  重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
5.  将新元素插入到该位置后
6.  重复步骤2~5

- java例子

```java
public void InsertSort(int[] array){
    if (array.length <= 0) return;
    for (int i = 1; i < array.length; i++){
        int j = i - 1;
        int key = array[i];
        while(j >= 0 && array[j] > key){
            array[j+1] = array[j];
            j--;
        }
        array[j+1] = key;
    }
}
```


## 选择排序
**时间复杂度：$O(n^2)$，最优时间复杂度：$O(n)$**

### 步骤：

首先在未排序序列中找到最小元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小元素，然后放到已排序序列的末尾。

- java例子

```java
public void SelectSort(int[] array){
    if (array.length <= 0) return;
    for (int i = 0; i < array.length; i++){
        int min = i;
        for (int j = i + 1; j < array.length; j++){
            if (array[min] > array[j]){
                min = j;
            }
        }
        swap(array, min, i);
    }
}
```



## 希尔排序
**时间复杂度：根据步长而不同，最优时间复杂度：O(n)**

### 步骤：

1.  比如下面的例子，对数组进行分区，按照步长进行分区，步长为4，分成四个区，用四个颜色表示

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/48327117.jpg" width=50%/>
</center>

2.  对每个分区应用插入排序，结果如下：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/33511557.jpg" width=50%/>
</center>

3.  再把步长缩减成1/2，再应用插入排序：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/11337902.jpg" width=50%/>
</center>

4.  直到步长为一

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/96792014.jpg" width=50%/>
</center>

## 堆排序
**时间复杂度：$O(nlogn)$，最优时间复杂度：$O(nlogn)$**

### 说明：

**堆（二叉堆**）可以视为一棵完全的二叉树

二叉堆一般分为两种：**最大堆**和**最小堆**

最大堆：

-   最大堆中的最大元素值出现在根结点（堆顶）
-   堆中每个父节点的元素值都大于等于其孩子结点（如果存在）

最小堆：

-   最小堆中的最小元素值出现在根结点（堆顶）
-   堆中每个父节点的元素值都小于等于其孩子结点（如果存在）

### 步骤：

先建立一个堆（最小或最大都可以）

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/78118630.jpg" width=50%/>
</center>

映射方式按照如下的公式进行：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/22158104.jpg" width=30%/>
</center>

然后进行堆调整，比如下图是按照最大堆的方式进行调整中的一趟：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/24421504.jpg" width=50%/>
</center>

取出堆顶元素，在重复进行以上步骤，直到只剩一个元素

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/94527342.jpg" width=50%/>
</center>


## 归并排序
**时间复杂度：$O(nlogn)$，最优时间复杂度：$O(n)$**

### 步骤：
归并操作（merge），也叫归并算法，指的是将两个已经排序的序列合并成一个序列的操作。归并排序算法依赖归并操作。可以用迭代或递归实现，主要采用分而治之的思想。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/46511354.jpg" width=50%/>
</center>

- 递归步骤：
  - 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
  - 设定两个指针，最初位置分别为两个已经排序序列的起始位置
  - 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
  - 重复步骤3直到某一指针到达序列尾
  - 将另一序列剩下的所有元素直接复制到合并序列尾
- 迭代步骤
  - 将序列每相邻两个数字进行归并操作形成$ceil(n/2)$个序列，排序后每个序列包含两/一个元素
  - 若此时序列数不是1个则将上述序列再次归并，形$ceil(n/2)$个序列，每个序列包含四/三个元素
  - 重复步骤2，直到所有元素排序完毕，即序列数为1
- java例子递归版

```java
public void mergeSort(int[] arr, int[] reg, int start, int end){
    if (start > end) return;
    int mid = (end + start) >> 1;
    int start2 = mid + 1;
    mergeSort(arr, reg, start, mid);
    mergeSort(arr, reg, start2, end);
    int k = start;
    while(start <= mid && mid + 1 <= end){
        reg[k++] = arr[start] < arr[start2] ? arr[start++] : arr[start2++];
    }
    while(start <= mid){
        reg[k++] == arr[start++];
    }
    while(start2 <= end){
        reg[k++] == arr[start2++];
    }
}
```

- java迭代版

```java
public void mergeSort(int[] arr){
    int len = arr.length;
    int[] reg = new int[len];
    int start, block;
    for(block = 1; block < len; block *= 2){
        for (start = 0; start < len; start += block * 2){
            int mid = start + block < len ? start + block : len;
            int high = start + 2 * block < len ? start + 2 * block : len;
            int low = start, start2 = mid;
            while (low < mid  && start2 < high){
                reg[low++] = arr[start] < arr[start2] ? arr[start++] : arr[start2++];
            }
            while(low <= mid){
                reg[low++] == arr[start++];
            }
            while(start2 <= end){
                reg[low++] == arr[start2++];
            }
        }
        int[] temp = arr;
        arr = reg;
        reg = temp;
    }
    reg = arr;
}
```

## 快速排序
**时间复杂度：$O(nlogn)$，最优时间复杂度：$O(nlogn)$**

使用二分查找的思路

### 步骤：

1.  从数列中挑出一个元素，称为"基准"（pivot），
2.  重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
3.  递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

- java例子

```java
public void Partition(int[] arr, int start, int end){
    if (start > end) return;
    int index = (int)(Math.random() * (end - start + 1)) + start;
    swap(arr, index, end);
    int small = start-1;
    for (index = start; index < end; index++){
        if (arr[index] < arr[end]){
            small++;
            if (small != index){
                swap(arr, index, small)
            }
        }
    }
    small++;
    swap(arr, small, end);
    return small;
} 
public void quickSort(int[] arr, int start, int end){
    if (arr.length <= 0) return;
    if (start == end) return;
    int index = Partition(arr, start, end);
    if (index > start){
        quickSort(arr, start, index - 1);
    }
    if (index < end){
        quickSort(arr, insex + 1, end);
    } 
}
```



## 剑指offer实例

### 面试题11：旋转数组的最小数字 
#### 题目描述
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
#### 解题思路



