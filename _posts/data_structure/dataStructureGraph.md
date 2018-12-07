---
title: 数据结构之图
date: 2018-04-18 21:04:38
tags:

-  数据结构
-  C++
categories: Data Structure
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/data_structure.jpg
---

# 图

## 基本术语
图G由两个集合V和E组成，记为$G = (V, E)$，V代表图中顶点的集合，E代表顶点之间的关系。将顶点的规模记为$n = |V|$，边集的规模记做$e = |E|$。

- 邻接（adjacency）顶点与顶点间的关系
- 关联（incidence）顶点与边的关系

- 无向图（undigraph）：邻接顶点u和v的次序无关系，则（u，v）为无向边。若图中都为无向边则为无向图。
- 有向图（digraph）：邻接顶点u和v的有头有尾，且u为尾（tail），v为头（head），则（u，v）为有向边。若图中都为有向边，则为有向图。
- 混合图（mixed graph）：一个图中既有有向边，又有无向边。
无向图可以用有向图表示。

路径：连续的边的端点构成的顶点序列。

- 简单路径：路径中不含重复的顶点
- 简单环路：除了起点和终点，其余顶点都不相同的环路。
- 欧拉环路：经过所有的边一次且恰好一次的环路
- 哈密尔顿环路：经过每一个顶点一次且恰好一次的环路

## 图的存储

### 邻接矩阵
使用二维数组表示顶点之间的相邻关系。设$G=(V, E)$是有n个顶点的图，顶点序号依次为0,1,···，n-1，则邻接矩阵可以表示为：
$arc\left [ i,j \right ]=\left\{\begin{matrix} 1 & \left ( v_{i},v_{j} \right ) \in E\cup \left \langle v_{i},v_{j} \right \rangle \in E\\ 0 & others \end{matrix}\right.$

### 关联矩阵
使用二位数字表示顶点和边的关系。设$G=(V, E)$是有n个顶点，e条边的图

### 顶点类
给出顶点类的一种实现方法，并未进行严格封装。其中`status`、`dTime`、`fTime`、`parent`、`priority`要进行重点分析。
```c++
typedef enum {UNDISCOVERED, DISCOVERED, VISITED} VStatus;
template <typename Tv> struct Vertex{   //顶点对象
    Tv data;                            //数据
    int inDegree, outDegree;            //入度，出度
    VStatus status;                     //状态
    int dTime, fTime;                   //时间标签
    int parent;                         //遍历树中的父节点
    int priority;                       //遍历树中的优先级
    Vertex(Tv const & d):               //构造新节点
        data(d),
        inDegree(0),outDegree(0),
        ststus(UNDISCOVERED),
        dTime(-1), fTime(-1),
        parent(-1),
        priority(INT_MAX){}   
};
```
#### 顶点操作
对于任意的顶点i，枚举出其所有的邻接顶点neighbor：
```c++
int nextNbr(int i, int j){             //表示若已经枚举到邻居j，则转向下一个邻居。采用逆序查找的方式，时间复杂度为O(n)
    while((-1<j)&&!exists(i,--j));     //短路求值，当j越界时跳出循环
    return j;
}

int firstNbr(int i){return nextNbr(i, n)} //返回首个邻居的索引，将n假想成一个尾哨兵。
```
如果改用邻接矩阵来查找，耗时将会降低到degree+1

#### 顶点插入
在邻接矩阵中，顶点插入对应于增加一列和一行，并且在对应的边集中增加其关联的边，在顶点集中增加该顶点。
```c++
int insert(Tv const & vertex){
    for(int i=0; j<n; j++){
        E[j].insert(NULL);                         //为每个列向量的末尾增加一个单元
    }
    n++;
    E.insert(Vector<Edge<Te>*>(n, n, NULL));       //边集增加边
    return V.insert(Vertex<Tv>(vertex));           //顶点集增加顶点
}
```

- 在每个列向量的尾部增加一个单元，初始化为NULL；
- 生成一个新的行向量，每个元素为一个边记录，总数为n，所有边引用都初始化为NULL。此处行向量的长度在原来的基础上增加1。
- 顶点集增加一个顶点元素。
这里未对新的边进行实质性操作。

#### 顶点删除
与顶点插入类似，需要删除顶点及其关联边，并返回该顶点信息。
```c++
Tv remove(int i){
    for(int j=0; j<n; j++){                      //删除所有出边
        if(exists(i, j)){
            delete E[i][j];
            V[j].inDegree--;
        }
    }
    E.remove(i);                                 //删除第i行
    n--;   
    for(int j=0; j<n; j++){                      //删除所有入边和第i列
        if(exists(j, i)){
            delete E[j].remove(i);
            V[j].outDegree--;
        }
    }
    Tv vBak = vertex(i);                         //备份顶点信息
    V.remove(i);                                 //删除顶点
    return vBak;
    
}
```


### 边类
对于Edge类也没有进行严格封装。
```c++
typedef enum {UNDTERMINED, TREE, CROSS, FORWARD, BACKWARD} EStatus;
template <typename Te> struct Edge{
    Te data;
    int weight;
    EStatus status;
    Edge(Te const & d, int w):
        data(d),
        weight(w),
        status(UNDTERMINED){}   
}
```
#### 边操作
```c++
bool exists(int i, int j){
    return (0<=i)&&(i<n)&&(0<=j)&&(j<n)&&E[i][j] != NULL;  //短路求值
}
Te & edge(int i, int j){
    return E[i][j]->data;
}
Estatus & status(int i, int j){
    return E[i][j]->status;
}
int & weight(int i, int j){
    return E[i][j]->weight;
}
```
判断边是否合法，如果合法可以将该边对应的数据取出，且该边的其他状态也可以直接返回，其时间复杂度为$O(1)$。

#### 边插入
使用邻接矩阵，将需要插入的边封装成一个边信息，再将地址链接到邻接矩阵对应的地方即可。
```c++
void insert(Te const & edge, int w, int i, int j){
    if(exists(i, j)) return;           //忽略已有的边
    E[i][j] = new Edge<Te>(edge, w);   //创建新的边并赋值给E[i][j]
    e++;                               //更新边数
    V[i].outDegree++;                  //更新关联顶点i的出度
    V[j].inDegree++;                   //更新关联顶点j的入度
}
```
#### 边删除
在邻接矩阵中删除一条边只需要将边插入的过程反过来。查询邻接矩阵，对应到一条边记录，删除边记录，使其查询时对应为空，返回删除边的信息。
```c++
Te remove(int i, int j){
    if(!exists(i, j)) return;           //忽略不存在的边
    Te eBak = edge(i, j);               //备份要删除的边
    delete E[i][j];
    E[i][j] = NULL;
    e--;
    V[i].outDegree--;
    V[j].inDegree--;
    return eBak;
}
```


### GraphMatrix
GraphMatrix派生于Graph类。首先构造顶点集和边集，其中顶点集是顶点构成的向量；边集相当于由边向量构成的矩阵，每个边向量长度为n，因此恰好构成邻接矩阵
```c++
template <typename Tv, typename Te> class GraphMatrix : public Graph<Tv, Te>{
    private:
        Vector<Vertex<Tv>> V;           //顶点集
        Vector<Vector<Edge<Te>*>> E;    //边集
    public:
        GraphMatrix(){                  //构造函数
            n = e = 0;
        }
        ~GraphMatrix(){                 //析构
            for (int j=0; j<n; j++){
                for (int k=0; k<n; k++)
                    delete E[j][k]
            }       
        }
}
```

#### 邻接矩阵表示优缺点

- 优点：
    - 直观，易于理解和实现
	- 适用范围广泛，尤其适用于稠密图（dense graph）
	- 判断两点之间是否存在联边只需要$O(1)$的时间
	- 获取顶点（出/入）度数只需要$O(1)$的时间
	- 添加或删除边后更新度数也只需$O(1)$的时间
	- 良好的扩展性（scalability）：
    	-  Vector良好的空间控制策略
    	-  可“透明处理”空间溢出的情况
- 缺点：
	- 消耗空间数与边数无关，总是会消耗$\Theta \left ( n^{2} \right )$的空间
		- 如平面图，其边数$e << n^{2}$，此时空间利用率约为$\frac{1}{n}$ 



## 算法
### 广度优先遍历
#### 过程
1. 访问顶点s
2. 依次访问s所有尚未访问的邻接顶点
3. 依次访问2步骤中的顶点的尚未访问的邻接顶点
4. ......
5. 直达没有尚未访问的邻接顶点

此算法会逐层访问顶点，灰色线条表示各邻接顶点之间可能会有的关系，但此算法会忽略这种关系。这种广度遍历是树的层次遍历的推广。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/17577110.jpg" width=75%/>
</center>

#### 算法
```c++
template <Typename Tv, TypenameTe>
void Graph<Tv, Te>::BFS(int v, int & clock){
    Queue<int> Q;
    status(v) = DISCPVERED;
    Q.enqueue(v);
    while(!Q.empty()){
        int v = Q.dequeue();   // 取出队首顶点v
        dTime(v) = ++clock;
        for(int u = firdtNbr(v); -1 < u; u = nextNbr(v, u)){  // 考察v的每个邻居u
            dealWithUforBFS(u, v)
            /***  视u的状态分别处理  ***/
        }
        status(v) = VISITED;
    }
}
```

遍历的起点是某个预先指定的顶点v。每个顶点都会经历从UNDISCOVERED状态到DISCOVERED状态，最后到VISITED状态，由此构成其生命周期。每个顶点都会入队和出队一次且仅一次，因此外层while循环的时间复杂度为$O(n)$，内层for循环是对顶点邻居的扫描，时间复杂度为$O(n)$，因此总体的时间消耗为$n^{2} + e$，时间复杂度为$O(n^{2})$，但在实际操作中，内层for循环的n很小，所以总体的时间消耗接近于$n + e$。对于邻居u的处理方式为：

```c++
template <Typename Tv, TypenameTe>
void dealWithUforBFS (Tv u, Tv v){
    if (UNDISCOVERD == status(u)){   // 若u尚未被发现，则发现该顶点
        status(u) = DISCOVERD;
        Q.enqueue(u);
        status(v, u) = TREE;         // 引入树边
        parent(u) = v;
    }else                            // 若u已经被发现（正在队列中）或已经访问完毕（已出队）
        status(v, u) = CROSS;        // 将（v,u）归类为跨边
}
```

并不是没幅图都只包含一个联通域，在有多个连通域的时候，从任何一个起点s出发，未必能够抵达其它的连通域，要使得bfs搜索足以覆盖整幅图而不是其中一个特定的连通域，需要使用while循环对BFS进行封装。

```c++
template <Typename Tv, TypenameTe>
void Graph<Tv, Te>::bfs(int s){            // s为起点
    reset();
    int clock = 0;
    int v = s;                             // 初始化，时间占用为θ（n + e）
    do                                     // 逐一检查所有顶点，一旦遇到尚未发现的顶点
        if(UNDISCOVERED == status(v))
            BFS(v, clock);                 // 从该顶点出发启动一次BFS
    while(s != (v = (++v % n)) );          // 按照序号访问，可不漏不重
}
```

#### 最短路径性

在树的结构中，相对于树根节点，任何一个节点v都对应于一条唯一的通路，这条路径的长度称作顶点v的深度，于是我们可以进而对所有的顶点自上而下按照它们的深度进行等价类划分，在每一个等价类中的所有顶点，所具有的深度指标都是彼此相等的。而树的层次遍历也相当于按照这一指标非降的次序，将所有的顶点逐一枚举出来，这样一个遍历的过程也可以转换为图结构的遍历。

图与树不同之处在于，从起始顶点s出发可能有多条路径都最后通往同一个顶点而且可能出现分叉。实际上只需考察顶点之间的最短通路，并且将这两个顶点之间的距离取作这条最短通路的长度。而在起始顶点相对固定的情况下，可以将s在这个记号中省掉，直接简称之为顶点v所对应的距离。

### 深度优先遍历

#### 过程：

- 访问顶点s
- if s的邻居尚未被访问，访问s尚未被访问的邻居，任取其一，递归执行DFS（u）
- else 返回

此过程等效于树的先序遍历，DFS会构造出原图的一颗支撑树。其过程为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/54619684.jpg" width=75%/>
</center>

按照图中的箭头方向，从红色到白色到黄色到蓝色，每次改变颜色都是因为处于else语句中。

#### 算法：

```c++
template <typename Tv, typename Te>
void Graph<Tv, Te>::DFS(int v, int & clock){
    dTime(v) = ++clock;
    status(v) = DISCOVERED;                                // 发现当前顶点
    for(int u = firstNbr(v); -1 < u; u = nextNbr(v, u)){   // 枚举v的每个邻居
        /*  视u的状态分别处理  */
        /* 与BFS不同，含有递归 */
        dealwithUforDFS(v, u)
    }
    status(v) = VISITED;
    fTime(v) = ++clock;
}
```

在内部对u进行处理时，没有使用队列的方式，而是涉及到递归调用。

```c++
template <typename Tv, typename Te>
void dealwithUforDFS(Tv v, Tv u){
    switch(status(u)){
        case UNDISCOVERED:                      // u尚未被发现，支撑树可以在此基础上扩展
            status(v, u) = TREE;
            parent(u) = v;
            DFS(u, clock);
            break;
        case DISCOVERED:                       // u已被发现但尚未访问完毕
            status(v, u) = BACKWARD;
            break;
        default:                               // u已被访问完，视承袭关系分为前向边或跨边
            status(v, u) = dTime(v) < dTime(u) ? FORWARD : CROSS;
            break;
    }
}
```


#### 无向图例子
例如对当前的无向图：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/85305440.jpg" width=75%/>
</center>

最终会得到一个支撑树：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/27991522.jpg" width=75%/>
</center>

与BFS（v）类似，在有多个连通域的时候，需要将DFS（v）用while循环封装起来

```c++
template <typename Tv, typename Te>
void Graph<Tv, Te>::dfs(int s){
    reset();                               // 初始化
    int clock = 0;
    int v = s;
    do                                    // 逐一检查所有顶点
        if(UNDISCOVERD = status(v))       // 一旦有尚未发现的顶点就从该点出发启动一次DFS      
            DFS(v, clock);
    while(s != (v = ++v % n));            // 按序号访问，不重不漏
}
```

在有向图中，情况更为复杂。一旦出现BACKWARD边，则表明图中出现了环路。
#### 括号引理
顶点活动期：`active[u] = (dTime[u], fTime[u])`
嵌套引理：给定有向图$G = (V, E)$及其任一DS森林，则

- u是v的后代，if and only if active[u]$\subseteq $active[v]
- u是v的祖先，if and only if activate[u]$\supseteq $activate[v]
- u与v无关，if and only if actiavte[u]$\cap$active[v] = $\varnothing$

祖先的活跃期必定包含后代的活跃期。借助时间标签可以在$O(1)$的时间内得出两个节点之间的祖先关系，若没有时间标签则需要从起点一直追溯到终点。