---
title: 数据结构之列表
date: 2018-04-05 23:05:39
tags:
- 数据结构
- C++
categories: Data Structure
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/data_structure.jpg
---
# 列表基础

对于数据结构的操作可以分为两类：

- 静态：仅读取get      $O(1)$、search    $O(logn)$操作
- 动态：需写入insert  $O(n)$、remove  $O(n)$操作
  列表采用动态存储策略：
- 列表元素称为节点（node）
- 各节点通过指针或引用彼此联接，形成一个线性序列
- 相邻节点彼此互称为前驱（predecessor）或后继（successor）


##  向量与列表的区别

- 向量：可根据元素的秩直接确定其物理地址。元素V[i]的物理地址为
  $V + i × s$
(s为单个单元占用的空间量)。寻秩访问效率较高。

- 列表：也可以进行循秩访问，但效率较低。应改为循位置访问（call-by-position）的方式，利用节点之间的互相引用，找到指定节点。


##  列表节点ADT接口

```
pred()                                  当前节点前驱节点位置
succ()                                  当前节点后继节点的位置
data()                                  当前节点所存的数据对象
insertAsPred(e)                         插入前驱节点，存入被引用对象e，返回新节点位置
insertAsSuccess(e)                      插入后继节点，存入被引用对象e，返回新节点位置
```
定义ListNode模板类：
```c++
#define Posi(T) ListNode<T>*
template <typname T>
struct ListNode{
    T data;
    Posi(T) pred;
    Posi(T) succ;
    ListNode() {};
    ListNode(T e, Posi(T) p = NULL, Posi(T) s = NULL)
        : data(e), pred(p), succ(s) {}
    Posi(T) insertAsPred(T const& e);
    Posi(T) insertAsSucc(T const& e);
}
```

接口操作：
```
size()
first(), last()                               返回首、末节点的位置
insertAsFirst(e), insertAsLast(e)             将e当做末节点插入
insertBefore(p, e), insertAfter(p, e)         将e当做节点p的直接前驱、后继插入
remove(r)
disordered()
sort()
find(e)                                        查找目标元素e
search(e)                                      查找目标元素e，返回不大于e且秩最大的元素
deduplicate()                                  删除重复元素
uniquify()                                     删除重复元素
traverse()                                     遍历向量并统一处理
```
在List结构中，为了便于理解，定义：

|        -1        |        0        |    ......     |      n-1      |        n        |      |
| :--------------: | :-------------: | :-----------: | :-----------: | :-------------: | ---- |
| header（头哨兵） | first（首节点） |    ......     | last(末节点)  | trailer(尾哨兵) |      |
|     固定存在     |   可能不存在    |    ......     |  可能不存在   |    固定存在     |      |
| 对外部invisible  |  对外部visible  | 对外部visible | 对外部visible | 对外部invisible |      |

# 无序列表

##  寻秩访问
无序列表可以进行循秩访问，可通过重载下标操作符进行，其时间复杂度为$O(r)$，效率低下。
```c++
template <typename T>
T List<T>::operator[](Rank r) const{
    Posi(T) p = first();
    while(0 < r--)
        p = p->succ;
    return p->data;
}
```

##  节点查找
当有多个重复元素的时候，会首先停止在最靠后的位置，在最坏情况下时间复杂度为$O(n)$
```c++
template <typename T>
Posi(T) List<T>::find(T const & e, int  n, Posi(T) p) const{
    while(0 < n--){                  //命中或越界才返回
        if(e == (p = p->pred)->data) //取出当前节点的数据域并与e比对
            return p;
    }
    return NULL;
}

find(e, n, p)         //在p的n个前驱中查找指定元素e
find(e, p, n)         //在p的n个后继中查找指定元素e
```

## 插入操作
当this指向的是首节点，这样操作会使插入的节点的前驱变为头节点。具体步骤为创建新节点时指定新节点的前驱和后继，即将当前节点前驱的后继指定为新节点，将当前节点后继的前驱指定为新节点。
```c++
template <typename T>
Posi(T) List<T>::insertBefore(Posi(T) p, T const& e){
    _size++;
    return p->insertAsPred(e);
}
template <typename T>
Posi(T) ListNode<T>::insertAsPred(T const& e){
    Posi(T) x = new ListNode(e, pred, this);  //首先创建一个ListNode，耗时
    pred ->succ = x;                          //创建连接
    pred = x;
    return x;
}

```
insertAsLast(e)   等价于  insertBefore(trailer, e)
基于复制的构造：
```c++
template <typename T>
void List<T>::copyNodes(Posi(T) p, int n){
    init();                                 //创建空的列表
    while(n--){ 
        insertAsLast(p->data);              //将从p开始的n项依次作为末节点插入
        p = p->succ;
    }
}
```

## 删除操作
在列表中删除指定元素时间复杂度为$O(1)$。
```c++
template <typename T>
T List<T>::remove(Posi(T) p){
    T e = p->data;              //备份删除节点的数据
    p->pred->succ = p->succ;    //待删除节点的后继 变为 待删除节点的前驱的后继
    p->succ->pred = p->pred;    //待删除节点的前驱 变为 待删除节点的后继的前驱
    delete p;
    _size--;
    return e;
}
```

销毁一个已有的列表(析构)，时间复杂度为$O(n)$，相当于反复执行remove操作。
```c++
template <typename T> 
List<T>::~List(){
    clear();                   //删除所有可见节点
    delete header;
    delete trailer;
}
template <typename T> 
int List<T>::clear(){
    int oldSize = _size;
    while(0 < _size){
        remove(header->succ)    //反复删除首节点，直到列表为空
    }
    return oldSize;
}
```

## 列表唯一化
将列表分为三部分：

- 已经没有重复元素的前面部分
- 当前查找的元素e
- 还未进行查找的后面部分
使用find操作从首节点遍历至末节点，在当前节点的前驱中查找与当前节点数据相同的节点。其时间复杂度为$O(n^{2})$。
```c++
template <typename T >
int List<T>::deduplicate(){
    if(_size < 2) return 0;
    int oldSize = _size;
    Posi(T) p = first();                    //初始化
    Rank r = 1;
    while(trailer != (p=p->succ)){          //遍历从首节点直至末节点
        Posi(T) q = find(p->data, r, p);    //在p的r个前驱中查找相同的元素，r即整个前缀的长度也就是第一部分的长度
        q ? remove(q) : r++;                //如果有就删除该元素，没有就r++
    }
    return oldSize - _size;
}
```

# 有序列表

## 列表唯一化
有序列表的唯一化比无序列表的耗时少，因为其有序，则只需要检测相邻节点的数据是否相同，其时间复杂度为$O(n)$。
```c++
template <typname T> 
int List<T>::uniquify(){
    if(_size < 2)
        return 0;
    int oldSize = _size;
    ListNodePosi(T) p = first();
    ListNodePosi(T) q;
    while(trailer != (q=p->succ)){       //从首节点遍历到尾节点
        if(p->data != q->data)           //若相邻节点互异则转入下一个区段
            p = q;
        else 
            remove(q);
    }
    return oldSize - _size;
}
```

## 列表查找
其平均时间复杂度为$O(n)$
```c++
template <typename T>  {
Posi(T) List<T>::search(T const & e, int n, Posi(T) p) const{
    while(0 <= n--){                     //对p的最近n个前驱，从右向左逐个比较
        if(((p = p->pred)->data) <= e)
            break;
    }
    return p;
}
```

# 列表排序

## 选择排序法（selection sort）
类似于冒泡排序法，将序列分成两部分

- 前半部分是无序子序列，但最大值不超多后半部分的最小值
- 后半部分是有序子序列

改进方法：

- 找到前部分的最大值
- 将最大值移到后半部分的最前端

对列表中起始于位置P的连续n个元素做选择排序：
```c++
template <typname T>    {}
void List<T>::selectionSort(Posi(T) p, int n){
    Posi(T) head = p->pred;         //头哨兵初始化
    Posi(T) tail = p;
    for(int i = 0; i < n; i++){     //尾哨兵初始化
        tail = tail->succ;
    }
    while(1 < n){
        insertBefore(tail, remove(selectMax(head->succ, n)));  //remove返回节点数据
        tail = tail->pred;
        n--;
    }
}

Posi(T) List<T>:selectMax(Posi(T), int n){
    Posi(T) max = p;
    for(Posi(T) cur = p; 1 < n; n--){    //遍历后续节点
        if(!lt((cur = cur->succ)->data, max->data)) 
            max = cur;
    }
    return max;
}
```
在选择最大元素时，!lt（not less than）意思是前者比后者不小，也就是>=，如果改为严格>算法会不稳定，遇到相同的max元素的时候无法将该元素移动到末尾。
insert和remove都需要动态分配空间，即new一个ListNode和delete，消耗时间，因此需要优化。该算法时间复杂度为$\theta \left ( n^{2} \right )$。

## 插入排序法（insert sort)
将列表看成两个部分：

- 前部分sorted
- 后部分unsorted

算法从左到右进行插值，仅使用$O(1)$的辅助空间，属于就地算法(in-place)，时间复杂度为$O(n^{2})$。
```c++
template <typename T>
void List<T>::insertionSort(Posi(T) p, int n){
    for(int r = 0; r < n; r++){
        insertAfter(search(p->data, r, p))   //search会返回在p的前面r个元素中不大于p对应的数据的最大值的位置，之后insert在那个位置之后
        p = p->succ;                         //p转向直接后继
        remove(p->pred);
    }
}
```

### 逆序对（inversion）
在插值排序的过程中，将当前元素插入到前缀中的合适位置时，当前元素和前缀中的后缀部分会构成i对逆序对，该值就是search所需的次数，所以总体的插值损耗时间为$O(I + n)$。此算法具有输入敏感(input-sensitive)特性。






