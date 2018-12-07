---
title: 数据结构之向量
date: 2018/4/4 21:56
tags:
- 数据结构
- C++
categories: Data Structure
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/data_structure.jpg
---
&emsp;&emsp;根据清华大学邓俊辉老师的课程整理记录。使用C++编程。


## 向量基础

### 向量概念

&emsp;&emsp;向量是数组的抽象与泛化，由一组元素按照现行次序封装而成。

- 各元素与[0,n)内的秩一一对应
- 元素类型不限于基本类型 
- 操作、管理、维护更简化、统一、安全
- 可更为敏捷参与复杂数据结构的控制与实现

&emsp;&emsp;向量ADT接口：

```
size()
get()
put(r,e)
insert(r,e)
remove(r)
disordered()
sort()
find(e)                 查找目标元素e
search(e)               查找目标元素e，返回不大于e且秩最大的元素
deduplicate()           删除重复元素
uniquify()              删除重复元素
traverse()              遍历向量并统一处理
```

## 无序向量

### 向量的动态扩容

&emsp;&emsp;copyfrom接口在copy向量时申请的容量是copy内容的两倍，因此可根据这种方式来进行扩容，其累计增容时间为$O(n^{2})$,分摊增益时间为$O(n)$

&emsp;&emsp;扩容代码：

``` c++
void Vector<T>::expand(){
    if(_size<_capacity) return;
    _capacity = max(_capacity,DEFAULT_CAPACITY);
    T* oldElem = _elem;
    _elem = new T[_capacity <<=1]; //容量加倍,移位
    for(int i = 0; i < _size; i++)
        _elem[i] = oldElem[i];
    delete [] oldElem;
}
```

### 向量插入
&emsp;&emsp;将向量的后继整体向后移位，空出一个位置后将其插入
&emsp;&emsp;算法实现：
``` c++
Rank Vector<T>::insert(Raank r, T const & e){
    expand()  //如有必要先考虑扩容
    for(int i = _size; i > r; i--)  //从后往前移位，若从前往后移可能出现覆盖的现象
        _elem[i] = _elem[i-1];
    _elem[r] = e;
    _size++;
    return r;
}
```

### 区间删除
&emsp;&emsp;将后继左移填补删除部分，时间复杂度为$O(n)$。
``` c++
int Vector<T>::remove(Rank lo,Rank hi){
    if(lo==hi) return 0;
    while(hi < _size) 
        _elem[lo++] = _elem[hi++];
    _size = lo;
    shrink();  //更新规模
    return hi - lo;
}
```
&emsp;&emsp;单元素删除可看做区间删除的特例$[r]=[r,r+1)$，时间复杂度为$O(n)$。若在进行区间操作时，叠加单元素操作进行区间操作会导致$O(n^{2})$的时间复杂度。
``` c++
T Vector<T>::remove(Rank r){
    T e = _elem[r];  //备份被删除的元素
    remove(r, r+1); //调用区间删除算法
    return e;
}
```

### 查找操作
&emsp;&emsp;查找操作都是从后向前查找匹配的元素，在最坏的情况下，其时间复杂度为$O(n)$，但在n给定的情况下其时间复杂度为$O(1)$。
``` c++
Rank Vector<T>::find(T const & e, Rank lo, Rank hi){
    while((lo < hi--) && (e != _elem[hi]));
    return hi; //hi<lo时查找失败，否则hi为命中元素的秩
}
```

### 去重操作
&emsp;&emsp;无序向量的唯一化需要用到去重操作。
&emsp;&emsp;while循环中find操作查找当前元素的前驱，remove对后继操作，则累计时间复杂度为$O(n^{2})$。
``` c++
int Vector<T>::deduplicate(){
    int oldSize = _size;
    Rank i = 1;        //初始值从_elem[1]开始
    while(i < _size){  //从前向后查询是否有与_elem[i]相同的元素，若有就删除
        (find(_elem[i], 0, i)) < 0 ? i++ : remove(i);
    }
    return oldSize - _size;
}
```

### 遍历操作
&emsp;&emsp;统一对各元素实施visit操作
``` c++
struct Increase{
    virtual void operator()(T & e){e++;}
}

void increase(Vector<T> & V){
    V.traverse(Increase<T>());
}
```


## 有序向量

### 唯一化
&emsp;&emsp;将无序向量有序化后便于操作，如无序向量的去重操作对应有序向量的唯一化操作，前者的时间复杂度为$O(n^{2})$。

- 有序序列，任意一对相邻元素顺序
- 无序序列，总有一对相邻元素逆序
- 相邻逆序对的数量可用于度量向量逆序的程度

&emsp;&emsp;检查各相邻元素是顺序或者逆序：

``` c++
int Vector<T>::disordered() condt{
    int n = 0;
    for(int i=1; i < _size; i++)
        n += (_elem[i-1] > _elem[i]);
    return n;
}
```
&emsp;&emsp;将无序向量的去重操作推广到有序向量更为高效。

- 有序向量去重法一
``` c++
int Vector<T>::uniquify(){
    int oldSize = _size;
    int i = 0;
    while(i < _size-1){
        (_elem[i] == _elem[i+1]) ? remove(i+1):i++;
    }
    return oldSize - _size;
}
```
&emsp;&emsp;此方法的时间复杂度和deduplicate相同，都为$O(n^{2})$。改进算法，将重复的元素批量删除。

- 有序向量去重法二
``` c++
int Vector<T>::uniquify(){
    Rank i = 0, j = 0;
    while(++j < _size){             //逐一扫描，直至末尾元素
        if(_elem[i] != _elem[j])    //找到两个不同元素后将第i+1元素用j元素覆盖
            _elem[++i] = _elem[j];
    }
    _size = ++i;
    shrink();     //如果有必要截除尾部多余元素
    return j-i;   
}
```
&emsp;&emsp;此方法的时间复杂度仅为$O(n)$



### 二分查找
&emsp;&emsp;随机选择使用二分查找还是Fibonacci查找。
``` c++
Rank Vector<T>::search(T const & e, Rank lo,Rank hi) const{
    return (rand() % 2) ? binSearch(_elem, e, lo, hi) : fibSearch(_elem, e, lo,hi);
}
```

&emsp;&emsp;补充：Fibonacci数递归公式为$fib(n)=fib(n-1)+fib(n-2)$，若直接实现公式，此方法的时间复杂度为$O(2^{n})$。
``` c++
int fib(n){
    return (2>n) ? n : fib(n-1) + fib(n-2);
}
```
&emsp;&emsp;优化方法为：进行动态规划，可以将时间复杂度降为$O(n)$。
``` c++
int fib(n){
    int f = 0;
    int g = 1;
    while(0 < n--){
        g = g + f;
        f = g - f;
    } 
    return g;
}
```

&emsp;&emsp;查找的语义约定：

- 若查找元素小于向量最小元素，则返回lo-1即左哨兵
- 若查找元素大于向量最大元素，则返回hi-1即右侧哨兵左邻


- A版本算法
&emsp;&emsp;减而治之，以任意元素 x = S[mi]为界，将待查找区间分为三部分，将mi取做向量的中点，且不考虑重复元素。
```c++
static Rank binSearch(T* A, T const & e, Rank lo, Rank hi){
    while(lo < hi){
        Rank mi = (lo + hi) >> 1;   //取出lo和hi的中点，>>表示右移一位
        if(e < A[mi])
            hi = mi;
        else if(A[mi] < e)
            lo = mi + 1;
        else
            return mi;
    }
    return -1;      //查找失败，返回-1
}
```
&emsp;&emsp;此方法的时间复杂度为$1.5log(n)$。左右分支的比较次数不等，但递归深度相同，左侧的比较次数较少，右侧的比较次数较多，因此可以调整递归深度，将左侧拉深，右侧变浅，从而减少平均比较次数。因此改进算法，将中点设置为$fib(k-1)-1$。

&emsp;&emsp;Fib查找：
```c++
static Rank fibSearch(T* A, T const & e, Rank lo, Rank hi){
    Fib fib(hi -lo);  //创建Fib数列
    while(lo < hi){
        while(hi - lo < fib.get())   //前向顺序查找，确定Fib(k)-1的轴
            fib.prev()
        Rank mi = lo + fib.get() -1;
        if(e < A[mi]) 
            hi = mi;
        else if(A[mi] < e)
            lo = mi + 1;
        else
            return mi;
    }
    return -1;
}
```
&emsp;&emsp;设平均查找长度为$a(\lambda )\times log(n)$，在λ在[0,1)范围内，使得平均查找长度有最优值，可以列出:
&emsp;&emsp;$a\left (\lambda \right ) \times log\left (n \right ) = \lambda \times [1 + a\left (\lambda \right ) \times log\left (n \lambda \right )]+ \left (1 - \lambda \right )\times \left [ 2 + a\left (\lambda \right ) \times log\left (n \left (1 - \lambda \right ) \right ) \right ]$
&emsp;&emsp;整理后可得$\lambda = \varphi = 0.6180339...$时，$a\left ( \lambda  \right )= 1.44420...$达到最优值。

- B版本算法
&emsp;&emsp;分为两个区间，mi的前驱和后继判断次数都为1。
```c++
template <typename T> static Rank binSearch(T* A, T const & e, Rank lo, Rank hi){
    while(1 < hi -lo){      //区间宽度为1时退出循环
        Rank mi = (lo + hi) >> 1;
        (e < A[mi]) ? hi = mi : lo = mi;    //区间划分为[lo, mi)、[mi, hi)
    }
    return (e == A[lo] ? lo : -1;)
}
```

- C版本算法
&emsp;&emsp;为了实现语义规定，在B版本基础上改进，为最终版本。
```c++
template <tempename T> static Rank binSearch(T* A, T const & e, Rank lo, Rank hi){
    while(lo < hi){         //区间宽度为0时退出循环
        Rank mi = (lo + hi) >> 1;
        (e < A[mi]) ? hi = mi : lo = mi +1;  //区间划分为[lo, mi)、(mi, hi]
    }
    return --lo;  //lo-1为不大于e的元素的最大秩 
}
```
&emsp;&emsp;中点的选取可以动态选取，根据$\frac{mi-lo}{hi-lo}\approx \frac{e - A\left [ lo \right ]}{A\left [ hi \right ] - A\left [ lo \right ]}$进行插值查找 ，最终的平均查找次数为$log\left ( log\left ( n \right ) \right )$。
&emsp;&emsp;对比普通查找，插值查找的优势不明显，在查找宽度极大或操作成本极高的情况下优势 较为明显。且该方法易受到干扰，需引入乘法及除法的额外计算。

- 最终可行方法
&emsp;&emsp;通过插值查找缩小查找范围，再进行二分查找。
&emsp;&emsp;查找方式选择：

-  大规模：插值查找
-  中规模：折半查找
-  小规模：顺序查找

## 无序向量有序化

&emsp;&emsp;向量元素有序排列时，计算效率会大大提升，如去重、查找等操作。

### 冒泡排序
```c++
template <typename T> void Veector<T>::bubbleSort(Rank lo, Rank hi){
        while( !bubble(lo, hi--));     //逐趟扫描交换，直至全序
}
template <typename T> bool Vector<T>::bubble(Rank lo, Rank hi){
        bool sorted = true;         //整体有序标志
        while (++lo < hi){           //逐一检查相邻元素，若为逆序则交换
               if(_elem[lo-1]>_elem[lo]){
                     sorted = false;
                     swap(_elem[lo-1], _elem[lo]);
               }
        }
        return sorted;
}
```
&emsp;&emsp;相应的java代码：
```java
public class DataStructure {
    private static boolean sorted = true;
    private static int[] testlist;
    public static void main(String[] args) {
        testlist = new int[]{1, 20, 15, 5, 16, 10, 22};
        int hi = testlist.length;
        int lo = 0;
        while(!bubble(lo, hi--) && hi > 0);
    }

    private static boolean bubble(int lo, int hi){
        while(++lo < hi){
            if (testlist[lo -1] > testlist[lo]){
                sorted = false;
                int change = testlist[lo];
                testlist[lo] = testlist[lo-1];
                testlist[lo-1] = change;
            }
        }
        for (int i=0;i<testlist.length;i++)
            System.out.print(testlist[i] + "\t");
        System.out.println();
        return sorted;
    }
}
```
&emsp;&emsp;打印结果：
```
1    15    5    16    10    20    22    
1    5    15    10    16    20    22    
1    5    10    15    16    20    22    
1    5    10    15    16    20    22    
1    5    10    15    16    20    22    
1    5    10    15    16    20    22    
1    5    10    15    16    20    22    
```
&emsp;&emsp;可以看出在第四次扫描的时候，所有的元素都已经就位，但算法仍然在进行扫描，因此可以根据此算法进行改进，将hi移动到已经就位的元素的开头。

### 并归排序
```c++
template <typename T>
void Vector<T>::mergeSort(Raank lo, Rank hi){
    if(hi - lo < 2)
        return;
    int mi = (lo + hi) >> 1;
    mergeSort(lo, mi);  //前半段排序
    mergeSort(mi, hi);  //后半段排序
    merge(lo, mi, hi);  //归并
}
```
&emsp;&emsp;二路归并：将两个有序的序列合并成一个有序序列，S[lo, hi) = S[lo, mi)+S[mi,hi)。时间消耗为O(n)。
```c++
template <typename T> void Vector<T>::merge(Rank lo, Rank hi){
    T* A = _elem + lo;  //合并后的向量为A[0, hi -lo) = _elem[lo, hi)
    int lb = mi -lo;    //前子向量B[0, lb) = _elem[lo, mi)，为复制的A前半部分的值
    T* B = new T[lb];
    for(Rank i = 0; i <lb; b[i] = A[i++]);
    int lc = hi - mi;   //后子向量C[0, lc) = _elem[mi, hi)，直接指向mi之后
    T* C = _elem + mi;
    for(Rank i = 0, j = 0, k = 0; (j < lb)||(k <lc);){
        if((j < lb) && (lc <= k || (B[j] <= C[k]))) //短路求值，当k值越界
            A[i++] = B[j++];
        if((k < lc) && (lb <= j || (C[k] < B[j])))
            A[i++] = C[k++];
    }
    delete [] B;    //释放临时空间B   
}
```
&emsp;&emsp;判断时可以进行精简：不用考虑C提前耗尽的情况，如果C提前耗尽，将B粘贴到A的末尾。
```c++
for(Rank i = 0, j = 0, k =0; j < lb;){
    if((k < lc) && (C[k] < B[j]))
        A[i++] = B[j++];
    if(lc <= k || (B[j] <= C[k]))
        A[i++] = C[k++];
}
```
&emsp;&emsp;java实现：
```java
testlist = new int[]{1, 4, 15, 5, 9, 10, 22};

    private static void mergeSort(int[] testlist){
        int mid = testlist.length / 2;
        int[] B = new int[mid];
        for (int i = 0; i < mid ;i++){
            B[i] = testlist[i];
            System.out.print(B[i] + "\t");
        }
        System.out.println();
        int[] C = new int[testlist.length - mid];
        for (int i = mid; i < testlist.length; i++) {
            C[i-mid] = testlist[i];
            System.out.print(C[i-mid] + "\t");
        }
        System.out.println();
        for (int i = 0, j = 0, k = 0; j < mid;){
            if (k < C.length && C[k] < B[j])
                testlist[i++] = C[k++];
            if (C.length <= k || B[j] <= C[k])
                testlist[i++] = B[j++];
//            else
//                testlist[i++] = C[k++];
        }
        for (int i = 0; i < testlist.length; i++) {
            System.out.print(testlist[i]);
        }
    }
```
&emsp;&emsp;输出结果：
```
1    4    15            //B
5    9    10    22    //C 
1    4    5    9    10    15    22     //testList  
```

