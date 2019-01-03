---
title: IDT算法论文笔记
date: 2019-01-02 21:26:35
tags:
- 计算机视觉
- Action Recognition
- 论文笔记
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# Action Recognition with Improved Trajectories论文笔记

论文原文地址[Action Recognition with Improved Trajectories](https://ieeexplore.ieee.org/abstract/document/6751553/)

前面一篇介绍了DT算法，iDT算法在此基础上进行改进，基本框架和DT算法相同，考虑了相机的运动，在相邻帧之间使用SURF特征和密集光流进行特征匹配，引入对背景光流的消除，使得特征更集中于对人的运动描述。特征正则化的方式及特征的编码方式也发生了改变。

## 相机运动估计

最重要的一处改进，通过相机运动估计来消除背景光流以及轨迹。首先看DT算法中在没消除背景干扰时的轨迹分布。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/IDT.png" width="50%">
</div>
由于相机在运动，所以背景上也有很多轨迹，人的轨迹会受到相机运动的影响，而这些信息与要识别的动作关系不大，属于干扰信息，因此希望能够识别并消除这些轨迹。实际上轨迹的运动也是通过计算光流信息进行计算的，因此需要通过估计相机运动，来消除背景区域的光流。

为了检测出相机的背景运动，假设相邻两帧的同形物（homography）是相关的，同形物排除了人类、车辆等独立移动的物体。由于相邻两帧图像之间变化很小，iDT算法假设相邻的两帧图像之间的关系可以用一个投影变换矩阵来描述，即后一帧图像是前一帧图像通过投影变换得到的。因此，估计相机运动的问题就变成了利用前后帧图像计算投影变换矩阵的问题。

1. 第一步是检测同形物需要找到连续帧之间的关系，使用了两种方法：

- 提取SURF特征并根据最近邻规则匹配；
- 从光流中采样运动矢量。这里使用基于多项式展开的高效光流法

两种方法是互补的，SURF特征主要为了提取斑点类型的特征，光流特征则是为了采集边角特征

2. 第二步，获得同形体后，使用RANSAC算法鲁棒C算法估计投影变换矩阵。具体操作为：

记$t$时刻和$t+1$时刻的灰度图像分别为$I_t$和$I_{t+1}$,用两张图像计算得到投影变换矩阵$H$（$I_{t+1} = H \times I_t$）。然后用$H$的逆对$I_{t+1}$进行变换（warp），即:

$$
I_{t+1}^{wrap} = H^{-1} \times I_{t+1}
$$

$I_{t+1}^{wrap}$为假设不存在相机运动时$t+1$时刻的图像；用$I_t$和$I_{t+1}^{wrap}$就可以计算得到优化后的光流。

对于DT来说，消除相机运动有两个好处：

- 对于HOF及MBH特征来说有很大的提升，用单一描述子的分类准确率比起DT中有很大的提高 
- 移除相机运动产生的轨迹，通过设置阈值，消除优化后的光流中位移矢量的幅值小于阈值的轨迹。

## 删除因人类行为导致的不一致匹配

图像中人的动作可能比较显著，人身上的匹配点对会使得投影矩阵的估计不准确。因此iDT算法中使用一个huaman detector检测人的位置框，并去除该框中的同形体。从而使得人的运动不影响投影矩阵的估计。iDT中使用的是当时效果最好的human detector，其文章为”Weakly supervised learning of interactions between humans and objects”。

## 轨迹特征

对于每个轨迹，都计算四个描述符（T、HOG、HOF、MBH），同DT相同，对特征点进行密集采样和跟踪。在相邻帧之间提取特征匹配，使用RANSAC方法校正homography。还使用human detector进行人体检测，删除包含人体的homography。然后用第$t$帧估计到的$t+1$帧的homography计算得到$I_{t+1}^{wrap}$，在$I_{t+1}^{wrap}$上计算运动描述符（HOF、MBH），HOG保持不变，对每帧图像都进行homography和$I_{t+1}^{wrap}$进行计算以防止误差的传播，得到描述符后，使用RootSIFT方法进行标准化，即进行L1的标准化。

## 特征编码

在DT中，使用BoF的方法对得到的描述符进行编码，这篇论文采用FV（Fisher Vector）进行编码。

### Fisher Vector

1. 对于一幅图像，提取$T$个描述子，每个描述子是$D$维的，可以用$X = \{ x_t \in \mathbb{R} ^{D} , t = 1,...,T\}$表示，假设$T$个描述子符合独立同分布（i.i.d），则有：

$$
p(X| \lambda) = \prod_{t=1}^{T}p(x_t| \lambda)
$$

取对数后可得：

$$
\mathfrak{L} (X| \lambda) = \sum _{t =1}^{T} log p(x_t | \lambda)
$$

用一组K个高斯分布的线性组合（即GMM混合高斯模型）来逼近这个分布，其参数即为$\lambda$。GMM模型可以用下式描述：

$$
p(x_t | \lambda) = \sum _{i=1}^{K} w_ip_i(x_t | \lambda _i)
$$

$p_i$为第$i$个基高斯分布：
$$
p_i(x_t | \lambda) = \frac{1}{(2 \pi)^{\frac{D}{2}} |\sum_i|^{\frac{1}{2}}} \cdot 
e^{-\frac{(x-\mu_i)'(x-\mu_i)}{2\sum_i}}
$$

此处$\lambda$在计算FV时是已知量，是预先通过GMM求解得到的。$\lambda = \{ \omega_i, \mu_i, \sum_i, i=1,...,K \}$为描述独立同分布的参数，其中

- $\omega_i$为系数，$w_i \geq 0$，$\sum_i \omega_i = 1$；
- $\mu_i$为均值
- $\sum_i$为标准差

在GMM中的目标为：求参数$\lambda$使得$p(X| \lambda)$有最大值，使得它确定的概率分布生成这些给定数据点的概率最大

2. 在计算前先定义占有概率，即特征$x_t$由第$i$个高斯分布生成的概率：

$$
\begin{equation}
\begin{aligned}
\gamma _t(i)  &= p(i | x_t, \lambda) \\
&= \frac{\omega_i p_i (x_t| \lambda)}{\sum_{j=1}^K w_j p_j (x_t | \lambda)}
\end{aligned}
\end{equation}
$$

对各参数求偏导：
$$
\begin{equation}
\begin{aligned}
&\frac{\partial \mathfrak{L}(X|\lambda)}{\partial \omega_i} = \sum_{t=1}^{T} [\frac{\gamma _t(i)}{\omega_i} - \frac{\gamma _t(1)}{\omega_1}] \quad for(i \geq 2) \\
&\frac{\partial \mathfrak{L}(X|\lambda)}{\partial \mu_i^d} = \sum_{t=1}^{T} 
\gamma _t(i)[\frac{x_t^d - \mu_i^d}{(\sigma_i^d)^2}] \\
&\frac{\partial \mathfrak{L}(X|\lambda)}{\partial \sigma_i^d} = \sum_{t=1}^{T} 
\gamma _t(i)[\frac{(x_t^d - \mu_i^d)^2}{(\sigma_i^d)^3} - \frac{1}{\sigma_i^d}] \\
\end{aligned}
\end{equation}
$$

注意此处的$i$是指第$i$个高斯分布，$d$是指$x_t$的第$d$维，得到的结果数目为 

- $\omega$：$K-1$个；
- 均值：$K*D$个；
- 标准差$K*D$个。

因此共有$(2D+1)*K-1$个偏导结果，这里的$-1$是由于$\omega_i$的约束。

3. 在计算完之后，还需要进行归一化。对三种变量分别计算归一化需要的fisher matrix的对角线元素的期望： 

$$
\begin{equation}
\begin{aligned}
& f_{\omega_i} = T(\frac{1}{\omega_i} + \frac{1}{\omega_1}) \\
& f_{\mu_i^d} = \frac{T_ {\omega _i}}{(\sigma_i^d)^2} \\
& f_{\sigma_i^d} = \frac{2T_{\omega_i}}{(\sigma_i^d)^2}
\end{aligned}
\end{equation}
$$

$T$为描述子的数目，最终归一化的FV结果为：
$$
\begin{equation}
\begin{aligned}
& f_{\omega_i} ^{-\frac{1}{2}} \frac{\partial \mathfrak{L}(X|\lambda)}{\partial \omega_i} \\
& f_{\mu_i^d} ^{-\frac{1}{2}} \frac{\partial \mathfrak{L}(X|\lambda)}{\partial \mu_i^d} \\
& f_{\sigma_i^d} ^{-\frac{1}{2}} \frac{\partial \mathfrak{L}(X|\lambda)}{\partial \sigma_i^d}
\end{aligned}
\end{equation}
$$
综上，基于Fisher Vector的图像学习的完整过程应该描述为下面几个步骤：

1. 选择GMM中K的大小；
2. 用训练图片集中所有的特征（或其子集）来求解GMM（可以用EM方法），得到各个参数；
3. 取待编码的一张图像，求得其特征集合；
4. 用GMM的先验参数以及这张图像的特征集合按照以上步骤求得其fv；
5. 在对训练集中所有图片进行2,3两步的处理后可以获得fishervector的训练集，然后可以用SVM或者其他分类器进行训练。

经过fisher vector的编码，提高了图像特征的维度，能够更好的用来描述图像。FisherVector相对于BOF的优势在于，BOF得到的是一个及其稀疏的向量，由于BOF只关注了关键词的数量信息，这是一个0阶的统计信息；Fisher Vector并不稀疏，同时，除了0阶信息，Fisher Vector还包含了1阶(期望)信息、2阶(方差信息)，因此Fisher Vector可以更加充分地表示一幅图片。

### iDT中的FV

- 用于训练的特征长度：trajectory+HOF+HOG+MBH=30+96+108+192=426维
- 用于训练的特征个数：从训练集中随机采样了256000个
- PCA降维比例：2，即维度除以2，降维后特征长度为213。先降维，后编码

- Fisher Vector中高斯聚类的个数K：K=256

编码完成后使用SVM分类。