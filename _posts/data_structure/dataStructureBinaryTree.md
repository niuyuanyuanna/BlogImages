---
title: 数据结构之二叉树
date: 2018-04-06 15:11:21
tags:
- 数据结构
- C++
categories: Data Structure
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/data_structure.jpg
---
# 二叉树

## 树

- 向量的search操作效率较高，如二分查找，其效率可以达到$log(n)$，但其动态操作，无论是插入和删除，其效率都较低，需要线性的时间。
- 列表的search操作，其效率较低，需要线性的时间，但由于其循位置访问的方式，一旦给定具体的操作位置，对于列表的动态操作就只需要在局部进行，时间损耗为$O(1)$。
- 二叉树结合了向量和列表的优点，可以理解为二维的列表。将树形结构称为半线性结构。

### 树的基本概念

1. 树： 在树型结构中，彼此元素之间的关系为edge，顶点为vertex，区别于列表中的node。需要为每一棵树指定一个特殊的顶点，称为根(root)。
2. 有根树：指定了其中一个顶点作为根的树。通过彼此的嵌套，小型的有根树可以逐步地整合为规模更大的有根树。
3. 兄弟树(sibling)：同一棵树的子树，它们之间根据度（degree）来度量。
4. 度数、顶点、边数间的关系：顶点的度数之和 = 边数 = 顶点总数-1。度数和顶点数同阶，因此在考虑算法时间复杂度时，以顶点数n作为参照。
5. 有序树：对有同一个父顶点的兄弟树编号，此时的这些兄弟树都是有序树。

### 树结构特性

#### 连通性 
k+1个节点通过k条边依次相连，构成一条路径。其路径长度=边数。
连通图（connected）：节点之间均有路径
#### 无环性 
当第1个节点和第k+1个节点相连时，构成一个环路（loop）
无环图（acyclic）：不含环路的图

#### 树特性

- 无环连通图
- 极小连通图
- 极大无环图

任一节点v与根之间存在唯一路径。因此一旦确定了根之后，所有的节点都可以根据这条唯一的路径定义一个参数depth，即v在这颗树中的深度。定义在path(v)上的节点为v的祖先（ancestor），v是它们的后代（descendent）。

- 半线性：在任意深度v的祖先 / 后代存在，则必然 / 未必唯一。

没有后代的节点称为叶子（leaf），所有叶子中深度最大的为树的高度，子树的高度也就是其根节点的深度。

### 树的表示
树结构的接口：
```
root()           根节点
parent()         父节点
firstChild()     长子
nextSibling()    兄弟
insert(i, e)     将e作为第i个孩子插入
remove(i)        删除第i个孩子（及其后代）
traverse()       遍历
```
由于每个节点只有一个父节的特性，在向上查询时，可以取得很好的效果，但在向下查询时需要遍历所有节点。在表格中增加子节点的信息，使得向下查找只需遍历当前节点的子节点。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/65826164.jpg" width=100%/>
</center>

但是这样操作对于子节点仍然有些多余，因此考虑将子节点查找的结构变为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/70722176.jpg" width=50%/>
</center>

每个节点均设置两个引用，横纵坐标方向为firstChild()，纵坐标方向为nextSibling()。

## 二叉树
二叉树是节点度数不超过2的树，同一节点的孩子和子树用左右进行区分表示为：

- lChild()     ——   lSubtree()
- rChiled()  ——   rSubtree()
这里已经隐含了树的有序性。

### 基本特性

- 所有深度为k的节点最多不超过$2^{k}$个
- 含有n个节点，高度为h的二叉树中，其数量关系为：$h<n<2^{h+1}$
  - 当$n=h+1$时，二叉树退化为一条单链
  - 当$n=2^{h+1}-1$时，得到满二叉树(full binary tree)即所有节点的度数都是2

对于二叉树而言，其宽度随高度呈指数增长，称为涨宽。

- 真二叉树：对于只有单分支或者叶子节点添加一个或2个对应数据为0的孩子，将这个二叉树变为一个满二叉树。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/51267035.jpg" width=80%/>
</center>

二叉树可以描述任意一颗树，将任意一棵树用长子-兄弟法表示。多出的兄弟节点可以用兄弟节点的子节点表示。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/13837723.jpg" width=30%/>
</center>

### 二叉树的实现
二叉树的基本组成单位是binary node。
BinNode模板类：

```c++
# define BinNodePosi(T) BinNode<T>*   //节点位置
template <typename T> struct BinNode{
    BinNodePosi(T) parent, lChild, rChild;
    T data;
    int height;  //高度
    int size();  //子树规模
    BinNodePosi(T) insertAsLC(T const &);                 //作为左孩子插入
    BinNodePosi(T) insertAsRC(T const &);                 //作为右孩子插入
    BinNodePosi(T) succ();                                //当前节点的直接后继
    template <typename VST> void travLevel(VST &);        //子树层次遍历
    template <typename VST> void travPre(VST &);          //子树先序遍历
    template <typename VST> void travIn(VST &);           //子树中序遍历
    template <typename VST> void travPost(VST &);        //子树后序遍历
}
```

BinNode接口实现：
```c++
template <typename T>
BinNodePosi(T) BinNode<T>::insertAsLC(T const & e){
    return lChild = new BinNode(e, this);   // this.lChild = null
}

template <typename T>
BinNodePosi(T) BinNode<T>::insertAsRC(T const & e){  //只需要常数时间
    return rChild = new BinNode(e, this);   // this.rChild = null
}

template <typename T>   
int BinNode<T>::size(){          //后代的总数是其根的子树之和。递归统计子树规模，需要线性时间
    int s = 1;
    if(lChild){
        s += lChild->size();
    }
    if(rChild){
        s += rChild->size();
    }
    return s;
}

```

BinTree模板：
```c++
template <typename T> class BinTree{
    protected:
        int _size;
        BinNodePosi(T) _root; //根节点
        virtual int updateHeight(BinNodePosi(T) x);  //更新节点x高度
        void updateHeightAbove(BinNodePosi(T) x);    //更新x及祖先高度
    public:
        int size() const{return _size;}              //国模
        bool empty() const{return !_root;}           //判断是否为空树
        BinNodePosi(T) root() const{return _root;}   //返回树根
        /*子树接入、删除、分离接口*/
        /*遍历接口  */
}
```

以高度更新接口为例：通过宏定义的封装方式高度，因为根据树的退化情况，其高度均不同。
```c++
# define stature(p) ((p) ? (p)->height : -1)
template <typename T>
int BinTree<T>::updateHeight(BinNodePosi(T) x){
    return x->height = 1 + max(stature(x->lChild), stature(x->rChild));   //采用常规二叉树规则，其时间复杂度为O(1)
}

template <typename T>
void BinTree<T>::udateHeightAbouve(BinNodePosi(T) x){//向上更新节点高度，其时间复杂度正比于树的深度。
    while(x){
        updateHeight(x);
        x = x->parent;
    }
}
```

节点插入：
在一个已有的树节点中，该节点原本没有右孩子。将新生成的一个节点插入到该节点的右侧，将其变为该节点的右孩子。
```c++
template <typename T> 
BinNodePosi(T)BinTree<T>::insertAsRC(BinNoidePosi(T) x){
    _size++;
    x->insertAsRC(e);
    updateHeightAbove(x);
    return x->rChild;
}
```
### 二叉树相关算法

#### 遍历
按照某种次序访问数中的各个节点，使得每个节点被访问恰好一次。遍历分为先序、中序、后序，按照当前节点与其左右孩子节点的访问次序来划分。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/10018835.jpg" width=90%/>
</center>

1. 先序遍历，时间复杂度为$O(n)$
```c++
template <typename T, typename VST>
void traverse(BinNodePosi(T) x, VST & visit){
    if(!x) return;  //先把树根节点取出访问再递归访问左右子节点
    visit(x->data);
    traverse(x->lChild, visit);
    traverse(x->rChild, visit);
}
```
可以将尾递归的形式化简为迭代的形式，引入一个栈存储节点的位置。
第一种迭代方法：
```c++
template <typename T, typename VST>
void travPre_I1(BinNodePosi(T) x, VST & visit){
    Stack <BinNodePosi(T)> S;
    if(x) S.push(x);
    while(!S.empty()){
        x = S.pop();
        visit(x->data);
        if(HasRChild(*x)) S.push(x->rChild); //右孩子先入后出
        if(HasLChild(*x)) S.push(x->lChild); //左孩子后入先出
    }
}
```
第一种迭代方法较难理解，不便于判断子节点的遍历次序，引入第二种迭代方法：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/51411613.jpg" width=90%/>
</center>

这种方法自上而下对做侧分支进行访问然后自下而上对右子树遍历。不同的右子树相互独立且自成一个子任务。

```c++
template <typename T, typename VST>
static void visitAlongLeftBranch(BinNodePosi(T) x, VST & visit, Stack <BinNodePosi(T)> & S){
    while(x){
        visit(x->rChild);   //访问当前节点
        S.push(x->rChild);  //右孩子入栈
        x = x->lChild;      //沿左侧链下行
    }
}

template <typename T, typename VST>
void travPre_I2(BinNodePosi(T) x, VST & visit){
    Stack <BinNodePosi(T)> S;
    while(true){
        visitAlongLeftBranch(x, visit, S);
        if(S.empty()) break;
        x = S.pop();      //弹出下一子树根
    }
}
```

visitAlongLeftBranch()这个函数会从当前节点一直访问左侧链，并自下而上的将右子节点存入栈中。主循环只需要重复执行这个函数直到栈为空退出。

2. 中序遍历
递归算法：
```c++
template <typename T, typename VST>
void traverse(BinNodePosi(T) x, VST & visit){
    if(!x) return;  
    traverse(x->lChild, visit);
    visit(x->data);
    traverse(x->rChild, visit);
}
```
根据观察可以发现中序遍历是从根节点开始沿左侧分支向下，直到找到最后一个左孩子，访问左孩子后再访问上层节点，最后访问该上层节点的右孩子。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/99511231.jpg" width=90%/>
</center>

迭代算法：
```c++
template <typename T>
static void goAlongLeftBranch(BinNodePosi(T) x, Stack <BinNodePosi(T)> & S){
    while(x){
        //visit(x->rChild);   //访问当前节点
        //S.push(x->rChild);  //右孩子入栈
        S.push(x);
        x = x->lChild;      //沿左侧链下行
    }
}

template <typename T, typename VST>
void trav_I1(BinNodePosi(T) x, VST & visit){
    Stack <BinNodePosi(T)> S;
    while(true){
        goAlongLeftBranch(x, S);
        if(S.empty()) break;
        x = S.pop();      //左侧节点访问
        visit(x->data);
        x = x->rChild;    //转向右子树，当右子树为空时进行处理。
    }
}
```
goAlongLeftBranch（）函数从当前节点出发，逐批将左侧节点逆序入栈，直到找到最左侧节点。在主函数中弹出栈顶节点，进行访问后再转向右子树。最终主程序的时间复杂度为$O(n)$。

3. 后续遍历
递归算法：
```c++
template <typename T, typename VST>
void traverse(BinNodePosi(T) x, VST & visit){
    if(!x) return;  
    traverse(x->lChild, visit);
    traverse(x->rChild, visit);
    visit(x->data);
}
```

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/66214575.jpg" width=90%/>
</center>

在后序遍历的过程中，从左侧链最后一个节点开始，再访问其右子节点，再自下而上访问其父节点。
迭代算法：
```c++
template <typename T>
static void gotoHLVFL(Stack <BinNodePosi(T)> & S){
    while(BinNodePosi(T) x= S.top()){
        if(HasLChild(*x)){
            if(HasRChild(*x)) S.push(x->rChild);
            S.push(x->lChild);
        }else
            S.push(x->rChild);
    S.pop();
}

template <typename T, typename VST>
void travPost_I(BinNodePosi(T) x, VST & visit){
    Stack <BinNodePosi(T)> S;
    if(x) S.push(x);
    while(!S.empth()){
        if(S.top()!=x->parent) gotoHLVFL(S);
        x = S.pop();      //左侧节点访问
        visit(x->data);
    }
}
```
4. 层次遍历
上面的三种遍历方式都存在子节点先于父节点接受访问的逆序情况，因此使用栈进行存储。对于层次遍历而言，所有子节点都应严格的后于父节点接受访问，即为顺序对每层访问，需要用到队列。
```c++
template <typename T, typename VST>
void BinNode<T>::travPost_I( VST & visit){
    Queue<BinNodePosi(T)> Q;
    Q.enqueue(this);
    while(!Q.empty()){
        BinNodePosi(T) x = Q.dequeue();          //取出队首节点并访问
        visit(x->data);
        if(HasLChild(*x) Q.enqueue(x->lChild));  //左孩子入队
        if(HasRChild(*x) Q.enqueue(x->rChild));  //右孩子入队
    }
}
```
每个节点入队、出队的操作恰好为1次，整体的时间复杂度为$O(n)$

#### 重构
已知二叉树的排列序列，还原出二叉树。
已知树的中序 + 先序/后序 即可还原出原始的二叉树。




