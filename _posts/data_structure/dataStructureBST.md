---
title: 数据结构之二叉搜索树
date: 2018-04-06 15:11:21
tags:
- 数据结构
- C++
categories: Data Structure
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/data_structure.jpg
---

# 二叉搜索树

## 概念
### 循关键码访问
数据各项之间，依照各自的关键码彼此区分。其中关键码需要满足以下条件

- 大小比较
- 相等比对

数据集合中的数据项统一表示和实现为词条（entry）形式
词条：
```C++
template <typename K, typename V> struct Entry{
    K key;
    V value;
    Entry(K k=K(), V v=V()):key(k), value(v){};   //默认构造函数
    Entry(Entry<K, V> const & e):key(e.key), value(e.value){};  // 克隆比较器、判断器
    bool operator< (Entry<K, V> const & e){return key < e.key}
    bool operator> (Entry<K, V> const & e){return key > e.key}
    bool operator== (Entry<K, V> const & e){return key == e.key}
    bool operator!= (Entry<K, V> const & e){return key != e.key}   
}
```

### BST
二叉搜索树（Binary Search Tree）首先是一颗二叉树，其次处处满足顺序性，即它的任一节点不小于其左后代或任一节点不大于其右后代。

- 顺序性：为局部的特征，但考察BST的中序遍历时可以发现它必然是单调非降的。
- 这一性质是BST的充要条件

例如下面的一颗BST

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/48098822.jpg" width=75%/>
</center>

根据中序遍历的顺序，列出访问的节点，可以看出其值为单调递增的。

#### BST模板

```c++
template <typename T> class BST:public BinTree<T>{
    public:          // virtual修饰，便于派生类重写
    	virtual BinNodePosi(T) & search(const T &);  // 查找
    	virtual BinNodePosi(T) insert(const T &);    // 插入
    	virtual bool remove(const T &);              // 删除
    protected:
    	BinNodePosi(T) _hot;                         // 命中节点的父亲
    	BinNodePosi(T) connect34(                    // 3+4重构
    		BinNodePosi(T), BinNodePosi(T), BinNodePosi(T),
    		BinNodePosi(T), BinNodePosi(T), BinNodePosi(T), BinNodePosi(T));
    	BinNodePosi(T) rotateAt(BinNodePosi(T));     // 旋转调整    
}
```

## 算法实现
### 查找
使用减而治之的方法，从根节点出发，逐步缩小查找范围，直到发现目标，或查找到空树，查找失败。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/73352075.jpg" width=75%/>
</center>

上面的例子需要查找到23，箭头方向表明了查找的路径，最后查找到了叶节点22，因此查找失败。整个过程可以视为在仿效有序向量的二分查找。

#### 实现方法一：

```c++
template <typename T>
BinNodePosi(T) & BST<T>::search(const T & e){
    return searchIn(_root, e, _hot = NULL);          // 从根节点启动查找
}
// 尾递归，可以更改为迭代操作
// v是当前树根节点
// e是目标关键码
// hot为记忆热点
static BinNodePosi(T) & searchIn(BinNodePosi(T) & v, const T & e, BinNodePosi(T) & hot){
    if(!v || (e == v->data))      // 当前子树已经为空则返回失败
    	return v;
    hot = v;
    return searchIn((e < v->data ? v->lChild : v->rChild), e, hot);
}
```

算法每递归一次，子树都会下降一层，因此其运行的时间正比于返回节点v的深度，但不超过树高，则耗时为：$O(h)$

#### 查找接口语义

- 返回值引用
  - 成功时：指向一个关键码为e且真实存在的节点，`_hot`指向返回命中节点的父亲
  - 失败时：指向最后一次试图转向空节点NULL，`_hot`指向最后一次转向的真实节点

空节点意思是其数值为NULL

- 在失败时，可以假象那个空节点为哨兵节点，并将其关键值假象为目标关键码
- 引入假象哨兵后，相当于返回值总是等效于命中节点，`_hot`总是指向命中节点的父亲

### 插入

#### 过程：

- 借助`search(e)`确定插入位置和方向，再将新节点作为叶子插入
- 若e不存在
  - `_hot`为新节点的父亲
  - `v = search(e)`为`_hot`对新孩子的引用
- 令`_hot`通过v指向新节点

#### 实现

```c++
template <typename T> BinNodePosi(T) BST<T>::insert(const T & e){
    BinNodePosi(T) & x = search(e);      // 查找目标
    if(!x){                              // 查找目标为空，即查找失败
        x = new BinNode<T>(e, _hot);     // 在x处创建新节点，并以_hot为父亲
        _size++;                         // 更新全树规模
        updateHeightAbouve(x);           // 更新x及历代祖先的高度
    }
    return x;
}
```

此算法主要的时间消耗在于`search(e)`和`updateHeightAbove(x)`两个函数都线性正比于返回节点x的深度，不超过树高`O(h)`。

### 删除

```c++
template <typename T> bool BST<T>::remove(const T & e){
    BinNodePosi(T) & x = search(e);      // 定位目标节点
    if(!x)                               // 忽略元素尚不存在的情况
    	return false;
    removeAt(x, _hot);
    _size--;
    updateHeightAbove(_hot);
    return true;
}
```

此算法在不考虑`removeAt(x, _hot)`时，时间主要消耗仍然在于`search(e)`和`updateHeightAbove(x)`，累计的时间消耗也是$O(h)$。接下来考虑`removeAt(x, _hot)`的情况

#### 情况一

如果删除的节点还有左孩子或者右孩子，只需要将对象删除，并且以它的子节点作为新进节点替代被删除的节点。这样可以保持BST的拓扑结构，也满足顺序性。

```c++
template <typename T> static BinNodePosi(T)
removeAt(BinNodePosi(T) & x, BinNodePosi(T) & hot){
    BinNodePosi(T) w = x;                   // 实际被删除的节点
    BinNodePosi(T) succ = NULL:             // 被删除节点的替代
    if(!HasLChild(*x))                      // 左子树为空
    	succ = x = x->rChild;
    else if(!HasRChild(*x))                 // 右子树为空
    	succ = x = x->lChild;
    else{                                   
        /******左右子树并存**********/
    }
    hot = w->parent;                        // 被删除节点的父亲
    if(succ)
    	succ->parent = hot;
    release(w->data);                       // 释放别删除节点
    release(w);
    return succ;                            // 返回替代节点
}
```

在这种情况下，只需要$O(1)$的时间。当左右孩子都为空时，`succ`指向`NULL`，上面的代码仍然正确。

#### 情况二

当左右孩子并存的时候，需要化繁为简。此处需要用到在二叉树中实现的一个接口`BinNode::succ()`该接口的作用是返回当前节点在中序遍历下的直接后继。找到之后将当前需要删除的节点和其直接后继调换位置，这时是一个中间状态，已经不再是一颗BST了。最后删除在直接后继位置处的目标节点，完成节点删除，重新变成一颗BST。

```c++
template <typename T> static BinNodePosi(T)
removeAt(BinNodePosi(T) & x, BinNodePosi(T) & hot){
    /*.........*/
    else{     // 左右子树均存在
        w = w->succ();
        swap(x->data, w->data);           // *x与其直接后继*w互换数据   
        BinNodePosi(T) u = w->parent;     // 原问题转换为 摘除直接后继
        (u == x ? u->rChild : u->lChild) = succ = w->rChild;
    }
    /*.........*/
}
```

其直接后继至多只有一个右孩子，因为作为直接后继，它一定是某条分支的左侧末端 。此时的时间消耗主要在于`succ()`其正比于x的高度，因此`search()`和`succ()`共不超过$O(h)$ 。

## 平衡等价

- BST主要接口的运行时间在最坏的情况下，线性正比于树高$O(h)$

- 在最坏的情况下，BST可能退化为一个列表，此时的查找效率会降为$O(n)$，线性正比于列表的规模

### 两种口径

1. 随机生成

对于n个互异的词条$\left \{ e_{1},e_{2}, ....., e_{n} \right \}$对任一排列$\sigma = \left \{ e_{i1},e_{i2}, ....., e_{in} \right \}$，从空树开始，反复调用`insert()`接口将各词条依次插入，得到$T(\sigma)$，与$\sigma$对应的$T(\sigma)$称为由$\sigma$随机生成的BST。

- 任一排列作为输入的概率均等，为$\frac{1}{n!}$
- 由n个互异词条随机生成的BST平均高度为$\Theta \left ( logn \right )$。

2. 随机组成

对于n个互异的词条，在遵循顺序性的前提下，可随机确定拓扑连接关系。由此所得的BST称为这组词条的随机组成。

- 由n个互异词条随机组成的BST，若共计$T(n)$棵，$T(n)=catalan(n)=\sum_{k=1}^{n}SP(k-1)\cdot SP(n-k)$。
- 所有BST等概论出现，其平均高度为$\Theta(\sqrt{n})$。

按照两种口径的平均性能，随机组成更为可信。因为在随机生成的过程中，不同的随机序列可能生成同一棵BST

###  两种平衡

1. 理想平衡

- 节点数目相对固定时，兄弟子树高度越接近平衡，全树也将倾向于更低。
- 由n个节点组成的二叉树，高度不低于$log_{2}n$，当正好等于$log_{2}n$时为理想平衡
- 理想平衡时相当于完全树甚至满树，此时条件太苛刻

2. 适度平衡

- 理想平衡出现概率极低，且维护成本过高，需要适当放松标准
- 高度渐进，不超过$O(logn)$，称为适度平衡
- 适度平衡BST称为平衡二叉搜索树（BBST)

### 等价BST

- 上下可变：连接关系不尽相同，承袭关系可能颠倒
- 左右不乱：中序遍历序列完全一致，全局单调非降

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/21182931.jpg" width=75%/>
</center>

如图这个例子可以看出其中序遍历序列完全相同，但其部分子序列拓扑结构并不相同。这样一对BST称为等价的BST。

对于各种BBST，将BST转换为BBST时，需要限制1. 单次动态修改操作后，至多$O(1)$处局部不再满足限制条件，2. 可以在$O(logn)$时间内，使得这些局部满足更新。

## AVL树

- AVL树是一种BBST，在AVL的标准下的平衡因子为`balFac(v) = height(lc(v)) - height(rc(v))`
- AVL树即是对任意的节点，其平衡因子都不超过1也不小于-1的BST。
- AVL树是适度平衡的，其高度不超过$O(logn)$

### 接口

```{
# define Balanced(x) (stature((x).lChild) == stature((x).rChild))  // 理想平衡
# define BalFac(x) (stature((x).lChild) - stature((x).rChild))     // 平衡因子
# define AvlBalabced(x) ((-2 < BalFac(x)) && (BalFac(x) < 2))      // AVL平衡条件
template <typename T> class AVL:public BST<T> {                    // 继承自BST
    public:                                                        // 沿用BSF::search()接口
    	BinNodePosi(T) insert(const T &);                          // 插入重写
    	bool remove(const T &);                                    // 删除重写
}；
```

在按照BST规则插入或删除节点后，AVL的平衡性会被破坏，因此需要借助等价变换

- 局部性：所有旋转都在局部进行 （每次仅需要$O(1)$时间）
- 快速性：在每一深度只需检查并旋转至多一次 （共$O(logn)$次）

#### 插入操作

1. 单旋插入

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/85915049.jpg" width=75%/>
</center>

   如上图所示，如果需要在v下面插入左孩子或者右孩子，则需要让g单旋调整，具体过程为：

   - 引入临时引用，指向节点p
   - 令p的左子树T1变为g的右子树
   - 令g为p的左孩子
   - 局部子树的根g替换为p

操作完成之后，局部子树的高度恢复，其更高的祖先也必然是平衡的，使得全树复衡。

2. 双旋插入

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/7993033.jpg" width=75%/>
</center>

同时可有多个失衡及诶单，最低者g不低于x的祖父，g经过双旋调整后复衡，子树高度复原，其更高的祖先也必然是平衡的，使得全树复衡。

其过程为：	

  - 围绕p顺时针旋转，`zig(p)`
      - 引入临时变量，指向节点v
      - 令v的右子树变为p的左子树
      - 令p为v的右孩子
      - 令g的右孩子为v
- 围绕节点g做一次逆时针旋转，`zag(g)`
  - 将临时变量指向节点v
  - 令v的左子树变为g的右子树
  - 令g为v的左孩子
  - 局部子树的根由g替换为v

##### 实现

```c++
template <typename T> BinNodePosi(T) AVL<T>::insert(const T & e){
    BinNodePosi(T) & x = search(e);
    if (x)
    	return x;
    BinNodePosi(T) xx = x = new BinNode<T>(e, _hot);     // 目标不存在则创建新节点
    _size++;
    // 从x的父亲_hot出发，逐层向上，依次检查各代祖先g
    for (BinNodePosi(T) g = _hot; g; g = g->parent){
        if (! AvlBalanced(*g)){                          // 一旦失衡就进行调整
            FromParentTo(*g) = rotateAt(tallerChild(tallerChild(g)));
            break;                                       // 完成对v，p，g的调整后就退出循环
        }else                                            // 未失衡，就更新其高度
        	updateHeight(g);
    }
    return xx;
}
```
#### 删除操作
1. 单旋删除

   在图中这种情况下，g、p、v是朝同一个方向排列。将T3的一个叶节点删除，会引起g点失衡。因此需要围绕点g进行一次zig操作。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/85320993.jpg" width=75%/>
</center>

   当T2最后的那个节点不存在，即调整后的子树高缩减了1，因此有可能引起更上一层的失衡，称为失衡传播现象，可能需要做$O(logn)$次调整。

2. 双旋删除

   在此图中，g、p、v并不是朝着同一个方向排列，此时删除T3的一个节点。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/69399379.jpg" width=75%/>
</center>

   这种情况下首先要围绕p做一次zag旋转，再围绕g做一次zig旋转。

   旋转完成后，情况和单旋类似，T1和T2这两棵子树可能其中一个会存在一个叶节点，那么旋转之后子树的高度缩减1，仍可能引起失衡，可能需要做$O(logn)$次调整。

##### 实现

```c++
template<typename T> bool AVL<T>::remove(const T & e){
    BinNodePosi(T) & x = search(e);
    if(!x)
    	return false;
    removeAt(x, _hot);                         // x存在则删除x
    size--;
    // 从_hot出发逐层向上，依次检查各代祖先
    for (BinNodePosi(T) g = _hot; g; g = g->parent){
        if (!AvlBalanced(*g))
        	g = FromParentTo(*g) = rotateAt(tallerChild(tallerChild(g)));
        updateHeight(g);
    }
    return true;
}
```

其中for循环可能要做$\Omega(logn)$次调整

### 3+4重构
设g(x)为最低的失衡节点，考察祖孙三代：g、p、v，按照中序遍历次序，将其重命名为$a < b < c$。则他们共拥有互不相交的四棵（可能为空）的子树，按照中序遍历次序命名为$T_{0}< T_{1} < T_{2} < T_{3}$。将原来以g为根的子树替换为一颗新的子树$S'$。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/64989590.jpg" width=75%/>
</center>

#### 实现

```c++
template <typename T> BinNodePosi(T) BST<T>::connect34(
	BinNodePosi(T) a, BinNodePosi(T) b, BinNodePosi(T) c,
	BinNodePosi(T) T0, BinNodePosi(T) T1, BinNodePosi(T) T2, BinNodePosi(T) T3,){
        a->lChild = T0;
        if (T0) 
        	T0->parent = a;
        a->rChild = T1;
        if (T1){
            T1->parent = a;
            updateHeight(a);
        }
        
        c->lChild = T2;
        if (T2) 
        	T2->parent = c;
        c->rChild = T3;
        if (T3){
            T3->parent = c;
            updateHeight(c);
        }
        
        b->lChild = a;
        a->parent = b;
        b->rChild = c;
        c->parent = b;
        updateHeight(b);
        
        return b;                  //返回子树新的根节点
	}
```

将`rotateAt()`完整化

```c++
template<typenmae T> BinNodePosi(T) BST<T>::rotateAt(BinNodePosi(T) v){
    BinNodePosi(T) p = v->parent;
    g = p->parent;
    if(IsLChild(*p)){                    // zig
        if(IsLChild(*v)){                // zig-zig
            p->parent = g->parent;
            return connect34(v, p, g, v->lChild, v->rChild, p->rChild, g->rChild);
        }else{                           // zig-zag
            v->parent = g->parent;
            return connect34(p, v, g, p->lChild, v->lChild, v->rChild, g->rChild);
        }
    }else{                               // zag
        if(IsRChild(*v)){                // zag-zag
            p->parent = g->parent;
            return connect34(g, p, v, g->lChild, p->lChild, v->lChild, v->rChild);
        }else{                           // zag-zig
            v->parent = g->parent;
            return connect34(g, v, p, g->lChild, v->lChild, v->rChild, p->rChild);
        }
    }
}
```

zig-zig 和zig-zag分别对应于：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/88247417.jpg" width=75%/>
</center>

剩下两种情况恰好与之相反。

### AVL树性能
优点： 

- 无论查找、插入或删除，最坏情况下的复杂度均为$O(logn)$，存储空间为$O(n)$

缺点：

- 引入平衡因子，需要改造元素结构或者进行额外封装

- 其实测性能和理论性能差距较大

  - 插入删除时的zig，zag成本高

  - 删除操作最多需要旋转$\Omega(logn)$次，若频繁插入或删除成本过高。、

- 单次动态调整后，全树拓扑结构变化量可能达到$\Omega(logn)$
