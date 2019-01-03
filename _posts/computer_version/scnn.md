---
title: SCNN论文笔记
date: 2018-12-24 22:21:03
tags:
- 计算机视觉
- Action Recognition
- 论文笔记
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# Temporal action localization in untrimmed videos via multi-stage cnns论文笔记

[Temporal action localization in untrimmed videos via multi-stage cnns](http://dvmmweb.cs.columbia.edu/files/dvmm_scnn_paper.pdf)是Zheng Shou发表在在CVPR2016上的论文，主要解决视频识别中的两个问题：

- Action Recognition：目的为判断一个已经分割好的短视频片段的类别。特点是简化了问题，一般使用的数据库都先将动作分割好了，一个视频片断中包含一段明确的动作，时间较短（几秒钟）且有唯一确定的label。所以也可以看作是输入为视频，输出为动作标签的多分类问题。常用数据库包括UCF101，HMDB51等。
- Temporal Action Location：不仅要知道一个动作在视频中是否发生，还需要知道动作发生在视频的哪段时间（包括开始和结束时间）。特点是需要处理较长的，未分割的视频。且视频通常有较多干扰，目标动作一般只占视频的一小部分。常用数据库包括THUMOS2014/2015, ActivityNet等。

这篇文章主要解决Temporal Action Localization的问题。SCNN指segment based CNN,即基于视频片段的CNN网络。

SCNN主要包括三个部分：

1. 多尺度视频片段生成
2. 动作分类网络
3. 分类网络微调

阅读论文之前可以阅读iDT论文



## 网络模型

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/SCNN.png" width="100%">
</div>

使用的每一段视频标注为$\Psi = \{  (\psi_m, \psi_m^{'} , k_m ) \}_{m = 1}^M $，其中$\psi_m$为动作起始时间，$\psi^{'}_m$为动作终止时间，$k_m$为该动作的id。$k_m \in \{ 1,2,...,K \}$，$K$为动作的种类。在训练过程中共有两种视频，一种是修剪过的视频$X \in \mathbb{T}$，对于修剪过的视频$\psi_m = 1$， $\psi_m^{'} = T$  ，$M=1$；另一种是未修剪的视频$X \in \mathbb{U}$

### 多尺度视频片段生成

第一步为生成候选的视频片段，作为下一阶段网络的输入。

首先将每一帧视频resize到$171*128$像素大小，对于未修剪过的视频，使用滑窗的方法产生视频片段，包括多个长度帧的片段：16,32,64,128,256,512。视频帧的重复度为75%。对于每一个未修剪视频，得到一系列视频片段：$\Phi=\{ (s_h, \phi_h, \phi_h^{'} ) \}^H_{h = 1}$，其中$H$是滑窗的个数，$\phi_h$和$\phi_h^{'}$是第$h$个片段$s_h$的起始时间和终止时间。

在得到视频片段后，对其进行平均采样16帧视频，从而使得输出的segment的长度均为16。

### C3D网络

C3D网络来自于[learning Spatiotemporal feature with 3DConvolutional Networks](https://arxiv.org/abs/1412.0767)这篇文章，前面的博客介绍过C3D网络及特征。

### Proposal Network

在生成训练数据时，同时还记录和segment和ground truth instance之间的最大重叠度（IoU)。对于proposal网络来说，将最大IoU大于0.7的标记为true，最大IoU小于0.3的标记为背景。以及类别（即如果存在多个重叠的ground truth,取重叠度最大的那个）。

将得到的所有片段输入到C3D网络中，经过fc8后分为两类，即判断是否为背景，训练时将IoU大于0.7的作为正样本（动作），小于0.3的作为负样本（背景），对负样本进行采样使得正负样本比例均衡。采用softmax loss进行训练。proposal network的主要作用是去除一些背景片段。

### Classification Network

经过Proposal Network后，背景被去除，对剩下的数据进行$K$个类别的动作分类。

和Proposal Network类似，经过fc8后输出$K+1$类，其中一类是背景，这个网络被用来初始化localization network, 仅在训练阶段使用，在测试阶段不使用。训练时同样将IoU大于0.7的作为正样本（K类动作），小于0.3的作为背景类，对背景类动作进行采样使得背景类动作的数量和K类动作数量的平均值相近。训练时同样采用softmax loss。

### Localization Network

在估计动作的起始位置时，需要抑制与groundtruth重叠部分很小的片段，增强重叠部分大的片段，如下面图中的B片段才是真正需要保留的，AC应该去除。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/LocalizationNetwork.png" width="60%">
</div>
为了达到这样的目标，在loss function增加了overlap的损失。在一个minibatch中，有N个训练样本$\{ (s_n, k_n, v_n) \}_{n = 1}^N$，对于第n个片段，fc8的输出向量为$O_n$，经过softmax后得到的预测结果为$P_n$，计算方式为：
$$
P_n^{(i)} = \frac{e^{O_n^{(i)}}}{\sum_{j=1}^N e^{O_n^{(i)}}}
$$
新的loss function为：
$$
L = L_{softmax} + \lambda \cdot L_{overlap}
$$
其中softmax损失函数为：
$$
L_{softmax} = \frac{1}{N} \sum_n(-log(P_n^{k_n}))
$$
是动作分类的损失函数，overlap的损失函数为：
$$
L_{overlap} = \frac{1}{N}\sum_n(\frac{1}{2} \cdot (\frac{(P_n^{k_n})^2}{(v_n)^\alpha} -1) \cdot [k_n > 0])
$$
这里的$k_n >0$表示$[k_n >0] = 1$时，该片段是动作，$[k_n >0] = 0$时，该片段是背景。$L_{overlap}$主要是为了增强与GT有很大重合度的片段的得分，抑制重合度较低的片段。

对$L_{softmax}$求导：
$$
\frac{\partial L_{softmax}}{\partial O_n^{(i)}} = 
\left\{\begin{matrix}
\frac{1}{N} \cdot (P_n^{(k_n)} - 1) & i = k_n\\ 
\frac{1}{N} \cdot P_n^{(i)} & i\neq k_n
\end{matrix}\right.
$$
对$L_{overlap}$求导：
$$
\frac{\partial L_{overlap}}{\partial O_n^{(i)}} =
\left\{\begin{matrix}
\frac{1}{N} \cdot (\frac{(P_n^{(k_n)})^2}{(v_n)^{\alpha}}) \cdot (1-P_n^{(k_n)}) \cdot [k_n > 0] & i = k_n\\ 
\frac{1}{N} \cdot (\frac{(P_n^{(k_n)})^2}{(v_n)^{\alpha}}) \cdot (-P_n^{(i)}) \cdot [k_n > 0] & i\neq k_n
\end{matrix}\right.
$$
==这里$i \neq k_n$的情况不知道是怎么得到的==

- 如果当前片段属于背景，则$L_{overlap} = 0$，此时的损失函数为$L = L_{softmax}$；
- 如果当前片段为动作，则$L$达到最小值时，取$\lambda = 1$，可计算得到$P_n^{(k_n)} = \sqrt{(v_n)^{\alpha}} $，因此惩罚两种情况：
  - 由于误分类，导致$P_n^{(k_n)}$很小；
  - 当$P_n^{(k_n)}$很大，超过目标$\sqrt{(v_n)^{\alpha}}$
- $L$随着$v_n$的减小而增大，表示与GT重叠部分小的片段不可靠，因为其可能包含很大的噪声。在特殊情况$v_n = 1$时，损失函数仅为softmax损失，随着$P_n^{(k_n)}$从0变为1，损失函数从$+ \infty$变为0。

### 预测阶段

预测时，滑动不同长度的时间窗，生成一组视频片段，输入到Proposal Network中，得到proposal的置信度得分$P_{prop}$，保留$P_{prop} > 0.7$的片段，将保留的片段通过Localization Network，得到动作类别及置信度$P_{loc}$，基于$P_{loc}$进行NMS去冗余检测。

