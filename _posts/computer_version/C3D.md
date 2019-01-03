---
title: C3D论文笔记
date: 2018-12-27 15:21:51
tags:
- 计算机视觉
- Action Recognition
- 论文笔记
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# learning Spatiotemporal feature with 3DConvolutional Networks论文笔记

C3D网络来自于[learning Spatiotemporal feature with 3DConvolutional Networks](https://arxiv.org/abs/1412.0767)这篇文章，发表于ICCV2015，主要贡献为三个方面：

1. 相对于2D卷积，3D卷积更适合学习时空特征
2. 对于3D卷积，所有卷积层均使用$3*3*3$的小卷积核计算效果更好
3. 将学习得到的特征命名为C3D，再与小的线性分类器组合，在4个不同的基准上优于现有的方法，并在其他2个基准上与目前最好的方法相当。

3D ConvNets在之前的文章就被提出，该方法使用人体检测器和头部跟踪来分割视频中的人物。分割后的视频作为3D ConvNets的输入来进行动作的分类。相比之下，本文将完整的视频帧作为输入，并且不依赖于任何预处理，容易地扩展到大型数据集。

## 3D ConveNets

这篇论文旨在探索3D ConvNets在大规模监督训练数据集和现代深度架构的情况下来获取不同类型视频分析任务的最佳表现。主要工作是如何构建3D ConvNets来学习时空特征，这一块主/要介绍了网络架构以及参数的设定、探索核的最佳时序深度、数据集以及训练的超参数设置。根据不同的任务，在不同测试集上进行了实验和对比，如Action recognition、Action Similarity Labeling（行为相似度标注）、Scene and Object Recognition。

### 网络结构

在3D ConvNets中，卷积和池化操作在时空上执行，而2D ConvNets中，仅在空间上完成。一张图像输入到2D卷积中，将输出一张对应的图像，若为多张图像，就可以将多张图像视为不同的通道，仍然输出一张对应图像，因此，2D ConvNets在每次卷积运算之后就会丢失时间信息。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/C3D.png" width="100%">
</div>

只有3D卷积才能保留输入信号的时间信息，从而产生输出，保留时间信息。

将输入的视频训练样本定义为：$c*l*h*w$，其中$c$是通道数，$l$是帧数，$h$ 、$w$分别为输入视频的高和宽，卷积核尺寸为$d*k*k$，其中$d$为卷积核深度，*k*为大小。网络输入视频的宽高调整为$3*16*128*171$。

网络由5个（卷积层 + 池化层）、2个全连接层、1个softmax层构成。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/C3DArchitecture.png" width="100%">
</div>
所有池化层采用max pooling且核的大小为$2*2*2$（第一层除外，为了不过早的合并时间信号）

### C3D特征描述

C3D特征可作为其他视频分析任务的特征。提取C3D特征的方法为：将视频分割为16帧长的片段，overlap为8帧，将视频片段输入到C3D网络中，提取fc6输出的4096维特征。C3D有选择性的提取到了外观和运动。C3D特征高效、简单、紧凑。