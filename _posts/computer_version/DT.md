---
title: DT算法论文笔记
date: 2018-12-19 21:30:37
tags:
- 计算机视觉
- Action Recognition
- 论文笔记
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# Dense Trajectories and Motion Boundary Descriptors for Action Recognition论文笔记

论文原文地址[Dense Trajectories and Motion Boundary Descriptors for Action Recognition](https://www.researchgate.net/publication/257672334_Dense_Trajectories_and_Motion_Boundary_Descriptors_for_Action_Recognition)

iDT算法是行为识别领域中非常经典的一种算法，在深度学习应用于该领域前也是效果最好的算法。由INRIA的IEAR实验室于2013年发表于ICCV。目前基于深度学习的行为识别算法效果已经超过了iDT算法，但与iDT的结果做ensemble总还是能获得一些提升。所以这几年好多论文的最优效果都是“Our method+iDT”的形式。这篇论文是iDT的基础。

## Dense Trajectories密集轨迹算法

通过光流场获取视频序列轨迹，沿轨迹提取形状特征、HOF、HOG、MBH特恒，再利用BoF方法对特征编码，最后训练SVM分类器。主要分为三个阶段：密集特征采样、特征点轨迹跟踪、基于轨迹的特征提取。其主要结构为：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/DT.png" width="90%">
</div>

### Dense Sampling密集采样

在多空间尺度通过网格划分的方式密集采样特征点，多空间尺度保证了采样特征点覆盖所有空间位置及尺度。通常取8个空间尺度。后续的特征提取在各个尺度上分别进行，特征点采样间隔$W$大概为5。

在时间序列跟踪特征点时，需要去除部分无法跟踪的特征点。通过计算每个像素点的自相关矩阵的特征值，并设置阈值$T$去除部分特征值
$$
T = 0.001 \cdot max_{i \in I} min(\lambda _i^1, \lambda_i^2)
$$
其中$\lambda_i^1, \lambda_i^2$为像素点$i$的特征值。

### Trajectory Shap Descriptor轨迹形状描述子

1. Dense Sampling得到的密集采样后得到的某特征点坐标为$P_t = (x_t, y_t)$，则下一帧图像的特征点位置$P_{t+1} = (x_{t+1}, y_{t+1})$为：

$$
P_{t+1} = (x_t, y_t) + (M * \omega_t)|_{x_t, y_t}
$$

$\omega_t = (u_t, v_t)$为密集光流场，由$I_t$，$I_{t+1}$计算得到

$M$为中值滤波器，大小为$3*3$

可以看出，计算特征点领域内的光流中值得到特征点的运动方向。

2. 连续$L$帧可以使用$P_t$计算出后面的$P_{t+1},...,P_{t+L}$帧的轨迹，但长时跟踪不可靠，每$L$帧需要重新进行密集特征点采样，一般$L=15$。
3. 轨迹本身也可以构成轨迹形状特征描述子。对于长度为$L$的轨迹，用$(\Delta P_t,..., \Delta P_{t+L-1})$描述，

$$
\Delta P_t = (P_{t+1} - P_t) = (x_{t+1} - x_t, y_{t+1} - y_t)
$$

正则化后获得轨迹描述子$T$:
$$
T = \frac{(\Delta P_t,..., \Delta P_{t+L-1})}{\sum _{j = 1} ^{t+L-1} ||\Delta P_j||}
$$

### Motion and Structure Descriptors运动和结构描述子

除了轨迹描述子$T$，还用到了HOG（Histogram of Gradient）、HOF（Histogram of Oriented Optical Flow）、MBH（Motion Boundary Histogram）等特征。

1. 沿着某特征点长度为$L$的轨迹，每帧图像取特征点周围$N \times N$区域（$N$=32），构成时空体（volume），对于时空体进行网格划分，空间上每个方向划分为$n_{\sigma}$份($n_{\sigma} = 2$)，时间上均匀选取$n_{\tau}$，($n_{\tau} = 2$)共分出$n_{\sigma} \times n_{\sigma} \times n_{\tau}$份。
2. 对各个区域进行特征提取特征

- **HOG**特征：计算灰度图梯度直方图，bin_num=8，输出长度为$2*2*3*8$
- **HOF**特征：计算光流直方图（包括方向和幅度信息），bin_num=8+1，额外一个用于统计光流幅度小于某阈值的像素点，输出长度为$2*2*3*9$
- **MBH**特征：计算的是光流图像梯度的直方图，也可以理解为在光流图像上计算的HOG特征，由于光流图像包括x方向和y方向，故分别计算$MBH_x$和$MBH_y$，输出长度为$2*2*2*3*8$

3. 在计算完后，还需要进行特征的归一化，DT算法中对HOG,HOF和MBH均使用L2范数归一化。

### Bag of Features特征编码

对得到的特征组（T，HOG，HOF，MBH）编码，得到一个定长的编码。

Bag of Features模型仿照文本检索领域的Bag-of-Words方法

- 把每幅图像描述为一个局部区域/关键点(Patches/Key Points)特征的无序集合。
- 使用某种聚类算法(如K-means)将局部特征进行聚类，每个聚类中心被看作是词典中的一个视觉词汇(Visual Word)，相当于文本检索中的词，视觉词汇由聚类中心对应特征形成的码字(code word)来表示（可看当为一种特征量化过程）。
- 所有视觉词汇形成一个视觉词典(Visual Vocabulary)，对应一个码书(code book)，即码字的集合，词典中所含词的个数反映了词典的大小。
- 图像中的每个特征都将被映射到视觉词典的某个词上，这种映射可以通过计算特征间的距离去实现，然后统计每个视觉词的出现与否或次数，图像可描述为一个维数相同的直方图向量，即Bag-of-Features。

在训练码书时，DT算法随机选取了100000组特征进行训练。码书的大小则设置为4000。在训练完码书后，对每个视频的特征组进行编码，就可以得到视频对应的特征。