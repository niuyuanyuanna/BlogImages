---
title: boosting算法
date: 2019-03-03 14:21:03
tags: 
- machine learning
- boosting
categories: Machine Learning
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/machine_learning.png
---


# Boosting算法汇总

## AdaBoost算法

考虑给定二分类的训练数据集：
$$
T = \lbrace { (x_1, y_1),(x_2,y_2),\ldots,(x_N,y_N) }\rbrace
$$
其中，每个样本点由实例与标记组成，实例$x_i \in \chi  \subseteq \mathbf{R} ^ n$，标记$y_i \in Y = (-1, +1)$，$\chi$为实例空间，$Y$为标记集合，AdaBoost算法从训练数据中学习一系列弱分类器或基分类器，并将这些弱分类器线性组合为一个强分类器。

### 过程

输入：

- 训练数据集：$T = \lbrace { (x_1, y_1),(x_2,y_2),\ldots,(x_N,y_N) }\rbrace$
- 弱学习算法

输出：

- 最终分类器$G(x)$

1. 初始化训练数据权值分布：

$$
D_1 = (w_{11}, w_{12},\ldots,w_{1i},\ldots,w_{1N}), \quad w_{1i} = \frac{1}{N}
$$

   假设训练数据集具有均匀的权值分布，即每个样本数据对于基本分类器的作用是相同的，保证在原始数据上学习基本分类器$G_1(x)$

2. 对 $m = 1, 2, ..., M$，即训练M个弱分类器

   1. 使用具有权值分布$D(m)$的训练数据集学习，得到基分类器：

$$
G_m(x):\chi \rightarrow (-1, +1)
$$

   2. 计算$G_m(x)$在加权训练数据集上的分类误差率:

$$
e_m = p(G_m(x_i) \neq y_i) = \sum_{i = 1} ^ N w_{mi}I(G_m(x_i) \neq y_i)
$$

   这里$w_{mi}$表示第m轮训练中第i个数据所占权重，且$\sum_{i = 1}^N w_{mi} = 1$，$G_m(x)$在加权的训练数据集上的分类误差是被误分类的样本的权值之和。

   3. 计算$G_m(x)$的系数$\alpha_m$：

$$
\alpha_m = \frac{1}{2}log \frac{1-e_m}{e_m}
$$

   $\alpha_m$表示$G_m(x)$在最终分类器$G(x)$中的重要性，由（5）式可以看出当$e_m \leq \frac{1}{2}$时，$\alpha_m >=0$，且$\alpha_m$随$e_m$减小而增大，因此可以看出分类误差率越小的基本分类器在最终的分类器中作用越大。

   4. 更新训练数据集的权值分布：

$$
D_{m + 1} =  (w_{m1}, w_{m2},...,w_{mi},...,w_{mN}) \\
w_{m + 1, i} = \frac{w_{mi}}{Z_{m}}e^{-\alpha_m y_i G_m(x_i)}\\
Z_m = \sum_{i = 1}^N w_{mi}e^{\alpha_m y_i G_m(x_i)}
$$

   其中$Z_m$为规范化因子，使得$D_{m+1}$成为一个概率分布。权值更新规则可以写为：

$$
w_{m + 1,i} = 
\begin{cases}
\frac{w_{mi}}{Z_{m}}e^{-\alpha_m} & G_m(x_i) = y_i\\ 
\frac{w_{mi}}{Z_{m}}e^{\alpha_m} &  G_m(x_i) \neq y_i
\end{cases}
$$

   可见，被误分类的权值被扩大，正确分类的权值被缩小。因此，误分类样本在下一轮的训练中起到更大的作用

3. 构建基本分类器的线性组合：

$$
f(x) = \sum _{m = 1}^M \alpha_m G_m(x)
$$
   这里$\sum_{m = 1} ^ M \alpha_m \neq 1$。最终的分类器模型为：

$$
G(x) = sign(f(x)) = sign(\sum_{m = 1} ^ M \alpha_m G_m(x))
$$

### 误差分析

AdaBoost算法的最终分类误差界为：

$$
\frac{1}{N} \sum_{i=1}^N I(G(x_i) \neq y_i) \leq 
\frac{1}{N} \sum_{i=1}^N e^{-y_i f(x_i)} = \prod_m Z_m
$$

对于二分类问题的分类误差界：

$$
\begin{aligned}
\prod_m Z_m &= \prod_m[2 \sqrt{e_m(1 - e_m)}] \\
&= \prod_m \sqrt{(1-4\gamma^2_m)} \\
& \leq e^{-2\sum_m \gamma^2_m}
\end{aligned}
$$

其中$\gamma_m = \frac{1}{2}-e_m$

若存在$\gamma > 0$，使得对所有的m有$\gamma_m \geq \gamma$，则：

$$
\frac{1}{N} \sum_{i=1}^N I(G((x_i) \neq y_i )\leq e^{-2M \gamma^2}
$$

表明在此条件下，训练误差以指数速率下降，且并不需要知道下界$\gamma$，其具有适应性。

### 前向分布算法和AdaBoost

AdaBoost模型为加法模型、指数损失、前向分布算法的二分类学习方法。它是前向分布加法算法的特例。

---

## 梯度提升（Gradient boosting）

Boosting、bagging和stacking是集成学习的三种主要方法。不同于bagging方法，boosting方法通过分步迭代（stage-wise）的方式来构建模型，在迭代的每一步构建的弱学习器都是为了弥补已有模型的不足。Boosting族算法的著名代表是AdaBoost。

### AdaBoost和GB区别

AdaBoost：

- AdaBoost算法通过给已有模型预测错误的样本更高的权重，使得先前的学习器学错的训练样本在后续受到更多的关注的方式来弥补已有模型的不足
- 经典的AdaBoost算法只能处理采用指数损失函数的二分类学习任务。
- AdaBoost算法对异常点（outlier）比较敏感

GB：

- 梯度提升方法在迭代的每一步构建一个能够沿着梯度最陡的方向降低损失（steepest-descent）的学习器来弥补已有模型的不足
- 梯度提升方法通过设置不同的可微损失函数可以处理各类学习任务（多分类、回归、Ranking等），应用范围大大扩展。
- 梯度提升算法通过引入bagging思想、加入正则项等方法能够有效地抵御训练数据中的噪音，具有更好的健壮性

机器学习中的学习算法的目标是为了优化或者最小化loss Function， Gradient boosting的思想是迭代生多个（M个）弱的模型，然后将每个弱模型的预测结果相加，后面的模型$F_{m+1}(x)$基于前面学习模型的$F_m(x)$的效果生成的，关系如下：

$$
F_{m+1}(x) = F_m(x) + h(x), \quad 1 \leq m \leq M
$$

如果目标函数是回归问题的均方误差，很容易想到最理想的$h(x)$应该是能够完全拟合$y - F_m(x)$，这就是常说基于残差的学习。残差学习在回归问题中可以很好的使用，但是为了一般性（分类，排序问题），实际中往往是基于loss Function 在函数空间的的负梯度学习，对于回归问题$\frac{1}{2}(y - F(x))^2$残差和负梯度也是相同的。

$L(y, f)$中的$f$，不要理解为传统意义上的函数，而是一个函数向量$f(x_1), \ldots, f(x_n)$，向量中元素的个数与训练样本的个数相同，因此基于Loss Function函数空间的负梯度的学习也称为“伪残差”。

### 过程

1. 初始化模型为常数值

$$
F_0(x) = \underset{\gamma}{argmin}\sum_{i = 1}^n L(y_i, \gamma)
$$

2. 对 $m = 1, 2, ..., M$，即训练M个弱分类器

   1. 计算伪残差

$$
\gamma_{mi} = -[\frac{\partial L(y_i, F(x_i))}{\partial F(x_i)}]_{F(x)=F_{m-1}(x)} \quad \mbox{for } i=1,\ldots,n
$$

   计算损失函数的负梯度在当前模型的值，将它作为残差估计。

   2. 基于$\gamma_{mi}$拟合一个回归树，得到m棵树的叶节点区域$R_{mj}$，$j=1,2, \ldots ,J$，以拟合残差的近似值。

   3. 对于$j=1,2,\ldots,J$，计算最优的$c_{mj}$

$$
c_{mj} = \underset{c}{argmin}\sum_{x_i \in R_{mj}}^n L(y_i, F_{m-1}(x_i) + c)
$$

   利用线性搜索估计叶节点区域的值，使损失函数极小化

   4. 更新回归树

$$
F_m(x) = F_{m-1}(x) + \sum_{j = 1}^Jc_{mj}I(x \in R_{mj})
$$

3. 得到回归树

$$
\hat{F}(x) = F_M(x) = \sum_{m = 1} ^ M \sum_{j = 1}^Jc_{mj}I(x \in R_{mj})
$$

---

## GBDT

GB算法中最典型的基学习器是决策树，尤其是CART，正如名字的含义，GBDT是GB和DT的结合。这里的决策树是回归树，GBDT中的决策树是个弱模型，深度较小一般不会超过5，叶子节点的数量也不会超过10，对于生成的每棵决策树乘上比较小的缩减系数（学习率<0.1），有些GBDT的实现加入了随机抽样（subsample 0.5<=f <=0.8）提高模型的泛化能力。通过交叉验证的方法选择最优的参数。因此GBDT的关键问题在于如何根据$\gamma_{mi}$拟合一个CART回归树。

### CART树

决策树可以认为是if-then规则的集合，易于理解，可解释性强，预测速度快。同时，决策树算法相比于其他的算法需要更少的特征工程，比如可以不用做特征标准化，可以很好的处理字段缺失的数据，也可以不用关心特征间是否相互依赖等。决策树能够自动组合多个特征，它可以毫无压力地处理特征间的交互关系并且是非参数化的，因此不必担心异常值或者数据是否线性可分（举个例子，决策树能轻松处理好类别A在某个特征维度x的末端，类别B在中间，然后类别A又出现在特征维度x前端的情况）。不过，单独使用决策树算法时，有容易过拟合缺点。所幸的是，通过各种方法，抑制决策树的复杂性，降低单颗决策树的拟合能力，再通过梯度提升的方法集成多个决策树，最终能够很好的解决过拟合的问题。由此可见，梯度提升方法和决策树学习算法可以互相取长补短，是一对完美的搭档。至于抑制单颗决策树的复杂度的方法有很多，比如限制树的最大深度、限制叶子节点的最少样本数量、限制节点分裂时的最少样本数量、吸收bagging的思想对训练样本采样（subsample），在学习单颗决策树时只使用一部分训练样本、借鉴随机森林的思路在学习单颗决策树时只采样一部分特征、在目标函数中添加正则项惩罚复杂的树结构等。

作为对比，先说分类树，CART是二叉树，CART分类树在每次分枝时，穷举每一个feature的每一个阈值，根据GINI系数找到使不纯性降低最大的的feature以及其阀值，然后按照feature<=阈值，和feature>阈值分成的两个分枝，每个分支包含符合分支条件的样本。用同样方法继续分枝直到该分支下的所有样本都属于统一类别，或达到预设的终止条件，若最终叶子节点中的类别不唯一，则以多数人的类别作为该叶子节点的性别。

回归树总体流程也是类似，不过在每个节点（不一定是叶子节点）都会得一个预测值，以年龄为例，该预测值等于属于这个节点的所有人年龄的平均值。分枝时穷举每一个feature的每个阈值找最好的分割点，但衡量最好的标准不再是GINI系数，而是最小化均方差--即（每个人的年龄-预测年龄）^2 的总和 / N，或者说是每个人的预测误差平方和 除以 N。这很好理解，被预测出错的人数越多，错的越离谱，均方差就越大，通过最小化均方差能够找到最靠谱的分枝依据。分枝直到每个叶子节点上人的年龄都唯一（这太难了）或者达到预设的终止条件（如叶子个数上限），若最终叶子节点上人的年龄不唯一，则以该节点上所有人的平均年龄做为该叶子节点的预测年龄。

### 过程

1. 初始化损失函数：

$$
f_0(x) = \underset{\gamma}{argmin}\sum_{i = 1}^n L(y_i, \gamma)
$$

2. 对 $m = 1, 2, \ldots, M$，即训练$M$个弱分类器

   1. 计算伪残差：

$$
\gamma_{mi} = -[\frac{\partial L(y_i, f(x_i))}{\partial f(x_i)}]_{F(x)=F_{m-1}(x)} \quad \mbox{for } i=1,\ldots,n
$$

   2. 对于$\gamma _{mi}$训练CART回归树，划分区域$R_{mj}$，$j = 1,2, \ldots ,J_m$

   3. 对于$j=1,2,\ldots,J_m$，计算最优的$\gamma_{mj}$:

$$
\gamma_{mj} = \underset{\gamma_{mj}}{arg min}\sum_{x_i \in R_{mj}} L(y_i, f_{m-1}(x_i) + \gamma)
$$

   4. 更新回归树：

$$
f_m(x) = f_{m-1}(x) + \sum_{j = 1}^{J_m} \gamma_{mj}I(x \in R_{mj})
$$


3. 得到最终的回归树：

$$
\hat{f}(x) = f_M(x) = \sum_{m=1}^M \sum_{j=1}^{j_m}\gamma_{mj}I(x \in R_{mj})
$$

---

## XGBoost

Xgboost是GB算法的高效实现，xgboost中的基学习器除了可以是CART（gbtree）也可以是线性分类器（gblinear）。

学习目标：假设共学习M棵树，则$\hat{y}_i = \sum_{m = 1}^M f_m(x_i)$

### 目标函数

xgboost在目标函数中显示的加上了正则化项，基学习为CART时，正则化项与树的叶子节点的数量T和叶子节点的值有关。

$$
L(\phi) = \sum_i l(\hat{y}_i, y_i) + \sum_m \Omega(f_m)
$$

其中

$$
\Omega(f) = \gamma T + \frac{1}{2}\lambda ||w||^2
$$


### 梯度提升

Gradient Boost使用Loss Function对$f(x)$的一阶导数计算出伪残差用于学习生成$f_m(x)$，xgboost不仅使用到了一阶导数，还使用二阶导数。

对于第m次的loss functiong:

$$
L_m = \sum_{i = 1}^n l(y_i, \hat{y}_{m-1,i} + f_m(x_i)) + \Omega(f_m)
$$

做二阶泰勒展开：$g$为一阶导数，$h$为二阶导数:

$$
L^{(m)} \simeq \sum_{i = 1}^n [l(y_i, \hat{y}_i^{(m-1)}) + g_i f_m(x_i) + \frac{1}{2}h_i f_m^2(x_i)] + \Omega(f_m) \\
g_i = \frac{\partial l(y_i, \hat{y}_i^{(m-1)})}{\partial \hat{y}^{(m-1)}} \\
h_i = \frac{\partial ^2 l(y_i, \hat{y}_i^{(m-1)})}{\partial \hat{y}_i ^2}
$$

当$l$为平方损失时，$g_i = 2(\hat{y}_i^{(m-1)}- y_i)$，$h_i = 2$。最终的目标函数只依赖于每个数据点的在误差函数上的一阶导数和二阶导数。由于之前的目标函数求最优解的过程中只对平方损失函数时候方便求，对于其他的损失函数变得很复杂，通过二阶泰勒展开式的变换，这样求解其他损失函数变得可行了。

重新定义每棵树为:

$$
f_m(x) = w_q(x)
$$

其中$q(x)$表示样本x在某叶子节点上，$w_q(x)$表示该节点的打分情况。把树拆分成结构部分$q$和叶子权重部分$w$。结构函数$q$把输入映射到叶子的索引号上去，而$w$给定了每个索引号对应的叶子分数

重新定义数的复杂度为：

$$
\Omega(f_m) = \gamma T + \frac{1}{2}\sum_{j=1}^T w_j^2
$$


其中$T$表示叶子节点数量，后面一想为权重正则化。

组合后的优化目标可以写为：

$$
\begin{aligned}
L^{(m)} &\simeq \sum_{i = 1}^n [ g_i f_m(x_i) + \frac{1}{2}h_i f_m^2(x_i)] + \Omega(f_m) \\
&= \sum_{i = 1}^n [ g_i f_m(x_i) + \frac{1}{2}h_i f_m^2(x_i)] + \gamma T + \frac{1}{2}\sum_{j=1}^T w_j^2 \\
&= \sum_{j=1}^T[(\sum_{i \in I_j} g_i)w_i + \frac{1}{2}(\sum_{i \in I_j} h_i + \lambda) w_j^2] + \gamma T
\end{aligned}
$$

定义$G_j = \sum_{i \in I_j} g_i$，$H_j = \sum_{i \in I_j} h_i $，$I_j = {(i|q(x_i) = j)}$表示每个叶子节点上样本的集合。

$$
\begin{aligned}
L^{(m)} &= \sum_{j=1}^T[(\sum_{i \in I_j} g_i)w_i + \frac{1}{2}(\sum_{i \in I_j} h_i + \lambda) w_j^2] + \gamma T \\
&= \sum_{j = 1}^T[G_jw_j + \frac{1}{2}(H_j+\lambda)w_j^2] + \gamma T 
\end{aligned}
$$


对$w_j$求导为0，得到：

$$
w_j^* = -\frac{G_j}{H_j + \lambda}
$$


这个式子代表指定一个树的结构时，在目标函数上最多可以减少多少（structure score），$L^{(m)}$的值越小，说明结构越好，这里的分数实际上是每个树的评分。对于每一次尝试去对已有的叶子加入一个分割 ：

$$
Gain = \frac{1}{2}[\frac{G_L^2}{H_L + \lambda} + \frac{G_R^2}{H_R + \lambda} - \frac{G_L^2+G_R^2}{H_L+H_R + \lambda}]-\gamma
$$

中间第一项表示切分后左子树得分，第二项为右子树得分，第三项为不切分时可以得到的分数，第四项表示增加新叶子节点引入的复杂度代价。对于分割点需要找到Gain较大的值进行分割。


## 总结

GBDT和XGBoost区别：

1. DBDT使用CART树为基分类器  ，loss优化时只用了一阶导数；XGBoost支持线性分类器，loss优化同时使用一阶导数和二阶导数。
2. XGBoost代价函数加入正则化项（叶子节点个数及叶子节点输出w的L2正则项），用于控制模型复杂度；
3. XGBoost支持特征抽样，一定情况下支持并行
