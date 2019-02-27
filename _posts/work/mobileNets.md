---
title: MobileNets
date: 2019-02-26 21:16:18
tags:
- 计算机视觉
- 实习项目
- 表情识别
categories: Work
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# MobileNets介绍

[MobileNets](https://arxiv.org/abs/1704.04861)是为移动和嵌入式设备提出的高效模型。MobileNets基于流线型架构(streamlined)，使用深度可分离卷积(depthwise separable convolutions,即Xception变体结构)来构建轻量级深度神经网络。

深度可分离卷积的作用：把标准卷积分解成深度卷积(depthwise convolution)和逐点卷积(pointwise convolution)。这么做的好处是可以大幅度降低参数量和计算量。分解过程如图：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/work/MobileNet.jpg" width=50%/>
</center>
主要思想是用上图中的（b）+（c）代替（a）。依然假设有$N$个卷积核，每个卷积核维度是$D_K \cdot D_K \cdot M$，输入feature map的通道数是$M$，输出feature map为$D_F \cdot D_F \cdot N$。那么（b）表示用$M$个维度为$D_K \cdot D_K \cdot 1$的卷积核去卷积对应输入的$M$个feature map，然后得到$M$个结果，而且这$M$个结果相互之间不累加（传统的卷积是用$N$个卷积核卷积输入的所有（也就是$M$个）feature map，然后累加这$M$个结果，最终得到$N$个累加后的结果），注意这里是用$M$个卷积核而不是$N$个卷积核，所以（b）中没有$N$，只有$M$。因此计算量是$D_K \cdot D_K \cdot M \cdot D_F \cdot D_F$。（b）生成的结果应该是$D_F \cdot D_F \cdot M$，图中的（b）表示的是卷积核的维度。

（c）表示用N个维度为1*1*M的卷积核卷积（b）的结果，即输入是$D_F \cdot D_F \cdot M$，最终得到$D_F \cdot D_F \cdot N$的feature map。这个就可以当做是普通的一个卷积过程了，所以计算量是$D_F \cdot D_F  \cdot 1  \cdot 1 \cdot M  \cdot N$（联系下前面讲的标准卷积是$D_K \cdot D_K \cdot M \cdot N \cdot D_F \cdot D_F$，就可以看出这个（c）其实就是卷积核为$1 \cdot 1$的标准卷积）。

## 标准卷积

标准卷积核会把输入的所有通道结果累加起来，得到一个卷积核的结果，共有N个卷积核，因此得到N个结果，输出$D_G \cdot D_G$就是一个$D_K \cdot D_K \cdot M$卷积所有通道累加后得到的结果。

输入特征F映射尺寸为$(D_F, D_F, M)$，采用标准卷积K的尺寸如图（a）所示，为$(D_K, D_K, M, N)$，输出的特征映射G尺寸为$(D_G, D_G, N)$。输入通道数为$M$，输出通道数为$N$。标准卷积计算公式为：
$$
G_{k,l,n} = \sum_{i,j,m}K_{i,j,m,n} \cdot F_{k+i-1, l+j-1, m}
$$
对应的计算量为：
$$
D_K \cdot D_K \cdot M \cdot N \cdot D_G \cdot D_G
$$

## 深度可分离卷积

将标准卷积拆分为深度卷积和逐点卷积后：

- 深度卷积负责滤波作用,尺寸为$(D_K,D_K,1,M)$如图(b)所示。输出特征为$(D_G,D_G,M)$;

深度卷积表示用$M$个维度为$D_K \cdot D_K \cdot 1$的卷积核卷积对应输入的$M$个$D_F \cdot D_F$的feature map，得到$M$个相互独立的结果，此时计算量为$D_K \cdot D_K \cdot M \cdot D_G \cdot D_G$，深度卷积得到的结果为$D_G \cdot D_G \cdot M$

- 逐点卷积负责转换通道，尺寸为$(1,1,M,N)$如图(c)所示。得到最终输出为$(D_G,D_G,N)$

逐点卷积表示用$N$个维度为$1 \cdot 1 \cdot M$的卷积核去卷积(b)的输出，为$D_G \cdot D_G \cdot M$，最终得到$D_G, D_G, N$，可理解为一个标准卷积，其计算量为$1 \cdot 1 \cdot M \cdot N \cdot D_G \cdot D_G$

拆分后，深度卷积公式为：
$$
\hat{G}_{k,l,n} = \sum_{i,j}\hat{K}_{i,j,m} \cdot F_{k+i-1, l+j-1, m}
$$
其中$\hat{K}$深度卷积，卷积核为$(D_K,D_K,1,M)$。上面公式表示第$m$个卷积核应用在$F$中第$m$个通道的输出。深度卷积和逐点卷积的计算量为：
$$
D_K \cdot D_K \cdot M \cdot D_G \cdot D_G + M \cdot N\cdot D_G \cdot D_G
$$

因此相对于标准卷积来说，计算量减少了到原来的
$$
\frac{D_K \cdot D_K \cdot M \cdot D_G \cdot D_G + M \cdot N\cdot D_G \cdot D_G}
{D_K \cdot D_K \cdot M \cdot N \cdot D_G \cdot D_G} = \frac{1}{N} + \frac{1}{D_K^2}
$$


## Depthwise Separable Convolution Util

卷积操作后都会跟一个Batchnorm和ReLU操作。上面的depthwise separable convolution是MobileNet的基本组件，在网络中，完整的depthwise separable convolution单元为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/work/DWSC.png" width=20%/>
</center>

## 网络结构

如果把depthwise和pointwise看做不同层的话，MobileNet一共包含28层。第一个卷积层不做分解，另外最后有个均值pooling层，全连接层和softmax层。这里dw就表示depthwiseMobileNet的网络结构为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/work/MobileNetModel.png" width=50%/>
</center>

在模型的加速和压缩方面有很好的前景。
