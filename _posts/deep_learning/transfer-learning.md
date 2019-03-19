---
title: 迁移学习--样本自适应
date: 2018-11-08 16:39:25
tags:
- CV
- transfer learning
categories: Deep Learning
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/deep_learning.jpg
---

# 迁移学习——样本自适应

## DaNN

2014年提出(Domain adaptive Neural Networks for Object Recognition)

### 网络结构

两层神经元组成：特征层和分类器层 
### 创新点
在特征层加入MMD适配层，用以计算源域和目标域的距离，网络的优化目标为：

- 在有label的源域数据上的分类误差$\ell_{C}$
- 对两个domain数据的判别误差$\ell_{D}$

因此，优化目标为：$\ell=\ell_{C} + \lambda \ell_{D}$，其中$\lambda$为网络适配权重参数。

### 缺点

由于网络太浅，表征能力有限，故无法很有效地解决domain adaptation问题（通俗点说就是精度不高）。
因此，后续的研究者大多数都基于其思想进行扩充，如将浅层网络改为更深层的AlexNet、ResNet、VGG等；
如将MMD换为多核的MMD等。

### MMD

最大均值差异（Maximum mean discrepancy），度量在再生希尔伯特空间中两个分布的距离，是一种核学习方法。

两个随机变量的距离为：  

$$
MMD[\mathfrak{F},X,Y]=[\frac{1}{m^{2}}\sum_{i,j=1}^{m}k(x_{i},y_{j}) - \frac{2}{mn}\sum_{i,j=1}^{m,n}k(x_{i},y_{j}) + \frac{1}{n^{2}}\sum_{i,j=1}^{n}k(x_{i},y_{j})]^{\frac{1}{2}}  
$$

其中

- $k()$是映射关系，类似于SVM中的核函数，把原变量映射到高维空间;
- X，Y为两种分布的样本，
- $\mathfrak{F}$表示映射函数集 

基于两个分布的样本，通过寻找在样本空间上的映射函数k，求不同分布的样本在k上的函数值的均值，通过把两个均值作差可以得到两个分布对应于k的mean discrepancy。寻找一个k使得这个mean discrepancy有最大值，就得到了MMD。
最后取MMD作为检验统计量（test statistic），从而判断两个分布是否相同。如果这个值足够小，就认为两个分布相同，否则就认为它们不相同。更加简单的理解就是：求两堆数据在高维空间中的均值的距离。
近年来，MMD越来越多地应用在迁移学习中。在迁移学习环境下训练集和测试集分别取样自分布p和q，两类样本集不同但相关。可以利用深度神经网络的特征变换能力，来做特征空间的变换，直到变换后的特征分布相匹配，这个过程可以是source domain一直变换直到匹配target domain。匹配的度量方式就是MMD。

## DDC(Deep Domain Confusion)

Deep Domain Confusion: Maximizing for Domain Invariance发表于2014年。DDC针对预训练的AlexNet（8层）网络，在第7层（也就是feature层，softmax的上一层）加入了MMD距离来减小source和target之间的差异。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/66547922.jpg" alt="DDC structure" title="DDC structure" width=50%/>
</div>

DDC在原有的AlexNet结构中，对网络的fc7添加一层适配器(Adaption layer)，单独考察网络对源域和目标域的判断能力。若判别能力很差就认为网络学到的特征不足以将两个领域数据区分开，有助于学习到对领域不敏感的特征表示。DDC是深度网络应用于迁移学习领域的经典作品。

## DAN(Deep Adaptation Networks)

DAN（2015）是在DDC的基础上发展而来，解决了DDC的两个问题：

- DDC只适配一层网络，效果不够好，因为不同层都是可以迁移的，因此DAN可以多适配几层
- DDC只用了单一核的MMD，这个核可能不是最优的核，因此采用多核的MMD，即MK-MMD。

DAN的网络结构为：
<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/12001851.jpg" alt="DAN structure" title="DAN structure" width=80%/>
</div>


### 多核MMD（Multi-kernel MMD）

MMD主要思想是把source和target用相同的映射方法映射到同一个再生核希尔伯特空间（RKHS）中，然后求映射后两部分的均值差异，作为两部分数据的差异。在MMD中这个核函数是固定的，在实现时可以选择是高斯核还是线性核使用单一的核函数，无法确定哪个核函数好。因此使用多个核构造总的核。

对于两个概率分布$p$，$q$，它们之间的MK-MMD为：

$$
d^2_k(p,q) \triangleq ||E_p[\phi(\mathbf{x}_s)]-E_q[\phi(\mathbf{x}_t)]||^2_{\mathcal{H}}
$$

多个核一起定义的kernel为：

$$
\mathcal{K} \triangleq \left\{k= \sum_{u=1}^{m}\beta_u k_u : \beta_u \ge 0, \forall u \right\}
$$

用$m$个不同的kernel加权，权重为$\beta_u$，得到的$\mathcal{K}$表征能力比单核更强。



### 多层适配

DDC方法中，只适配AlexNet的第七层，DAN仍然基于AlexNet网络，适配最后三个全连接层。因为网络的迁移能力在这三层会task-spacific，类似于Inception、ResNet等，在迁移学习时，只学习最后一层全连接层的参数，经过softmax层后得到分类信息。



### 优化目标

DAN方法，基于AlexNet网络，探索source和target之间的适配关系。任何一个方法都有优化的目标。DAN也不例外。它的优化目标由两部分组成：损失函数和分布距离。基本上所有的机器学习方法都会定义一个损失函数，它来度量预测值和真实值的差异。分布距离就是面提到的MK-MMD距离。于是，DAN的优化目标就是：

$$
\min_\Theta \frac{1}{n_a} \sum_{i=1}^{n_a} J(\theta(\mathbf{x}^a_i),y^a_i) + \lambda \sum_{l=l_1}^{l_2}d^2_k(\mathcal{D}^l_s,\mathcal{D}^l_t)
$$

- $\Theta$表示所有权重和bias参数，是需要学习得到的目标参数；
- $l_1$、$l_2$表示网络适配从第六层到第八层，前面的网络不进行适配；
- $\mathbf{x}_a$，$n_a$表示source和target中所有的label数据集合；
- $\lambda$为惩罚系数；
- $J(.)$定义一个损失函数，一般使用cross-entropy。

损失函数的前面部分是网络参数$\Theta$，后面部分为MMD的距离参数$\beta$

#### 网络参数学习

对$\Theta$的学习依赖于MK-MMD距离的计算。通过kernel trick（类比于以前的MMD距离）总是可以把MK-MMD展开成一堆内积的形式。然而，数据之间两两计算内积是非常复杂的，时间复杂度为$O(n^2)$，这个在深度学习中的开销非常大。
因此，提出对MK-MMD的无偏估计：

$$
d^2_k(p,q)=\frac{2}{n_s}\sum_{i=1}^{n_s/2}g_k(\mathbf{z}_i)\\
\mathbf{z}_i \triangleq (\mathbf{x}^s_{2i-1},\mathbf{x}^s_{2i},\mathbf{x}^t_{2i-1},\mathbf{x}^t_{2i})
$$

将kernel作用到$\mathbf{z}_i$上，变为：

$$
g_k(\mathbf{z}_i) \triangleq k(\mathbf{x}^s_{2i-1},\mathbf{x}^s_{2i})+k(\mathbf{x}^t_{2i-1},\mathbf{x}^t_{2i})-k(\mathbf{x}^s_{2i-1},\mathbf{x}^t_{2i})-k(\mathbf{x}^s_{2i},\mathbf{x}^t_{2i-1})
$$
只计算了连续的一对数据的距离，再乘以2，这样就可以把时间复杂度降低到$O(n)$。在具体进行SGD的时候，需要对所有的参数求导：对$\Theta$求导。在实际用multiple-kernel的时候，作者用多个高斯核。


## Beyond Sharing Weights for Deep Domain Adaptation

发表于2016年，提出了在适配层中target和source不共享参数的思想。与以往的Domain adaptation在Deep Learing中的运用不同，这篇论文提出，在source domain和target domain之间使用不同的参数而非共享参数（不是所有的layers都不共享参数，有些层还是共享了）。他们提出在试验中这种网络的表现会好于那些使用共享参数的网络。

其网络结构为双流结构：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/65970249.jpg" alt="BSW for DDA" title="BSW for DDA" width=70%/>
</div>

该结构引入loss防止两个分支对应的权重差异过大。


## Deep CORAL: Correlation Alignment for Deep Domain Adaptation

这篇文章发表于2016年，主要提出一个CORAL loss，通过对source domain和target domain进行线性变换将各自的二阶统计量对齐。

$$
L_{CORAL} = \frac{1}{4d^2} ||C_S - C_T||^2 \\
C_S = \frac{1}{n_S - 1}(D_S^TD_S - \frac{1}{n_S} (1^TD_S)^T (1^TD_S)) \\
C_T = \frac{1}{n_T - 1}(D_T^TD_T - \frac{1}{n_T} (1^TD_T)^T (1^TD_T))
$$
其中

- $n_S$，$n_T$为source domain和target domain的batch size；
- $d$为特征的维度；
- $1^T$为一个全1的向量

其网络结构为：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/32435164.jpg" alt="Deep CORAL" title="Deep CORAL" width=80%/>
</div>


## Deep Domain Adaptation by Geodesic Distance Minimization

文章发表于2017年，在CORAL loss的基础上将其改进为Log-CORAL loss。表示两个协方差矩阵的log之间的欧氏距离。公式为：

$$
L_{LogCORAL} = \frac{1}{4d^2} ||log(C_S) - log(C_T)||^2 \\
$$

其网络结构为：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/24542894.jpg" alt="Log-CORAL loss" title="Log-CORAL loss" width=80%/>
</div>

特征迁移目前都是小网络进行尝试，常用的特征分布的度量标准是MMD和CORAL等，但其在网络中的位置，以及损失核K的选取都很关键，并且发现多应用于FC层，哪些层应该或不应该共享其权重的最佳选择取决于实际的应用。 

FC可在模型表示能力迁移过程中充当“防火墙”的作用。具体来讲，假设在ImageNet上预训练得到的模型为M ，则ImageNet可视为源域（迁移学习中的source domain）。微调（fine tuning）是深度学习领域最常用的迁移学习技术。针对微调，若目标域（target domain）中的图像与源域中图像差异巨大（如相比ImageNet，目标域图像不是物体为中心的图像，而是风景照），不含FC的网络微调后的结果要差于含FC的网络。因此FC可视作模型表示能力的“防火墙”，特别是在源域与目标域差异较大的情况下，FC可保持较大的模型capacity从而保证模型表示能力的迁移。（冗余的参数并不一无是处。）

## 多领域适应

《Deep Cocktail Network: Multi-source Unsupervised Domain Adaptation with Category Shift》发表于2018年。