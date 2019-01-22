---
title: R-C3D论文笔记
date: 2019-01-03 22:18:24
tags:
- 计算机视觉
- Action Recognition
- 论文笔记
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# R-C3D: Region Convolutional 3D Network for Temporal Activity Detection论文笔记

[R-C3D: Region Convolutional 3D Network for Temporal Activity Detection](https://arxiv.org/abs/1703.07814)发表于ICCV2017，以C3D网络为基础，结合Faster R-CNN物体检测方法，对于任意输入的L帧视频，先用Faster R-CNN找到人体proposal，再进行3D-conv，最后进行回归和分类。文章的主要贡献为：

1. 可以对任意长度视频、任意长度行为结合proposal和分类进行端到端的检测；
2. 通过共享Progposal generation 和Classification网络的C3D参数提升网络速度；
3. 在三个测试集上测试并取得了不错的效果。

## 网络结构

R-C3D将目标检测方法faster R-CNN的RPN网络应用到人体检测中，并且将PoI pooling推广到3D维度，并进行端到端的训练。其网络结构如下：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/R-C3D.png" width="100%">
</div>
整个网络分为四个部分：

1. 特征提取网络：对于输入任意长度的视频进行特征提取
2. 候选框子网络：提取可能存在行为的时序片段
3. 动作分类子网络：对检测到的行为时序进行分类
4. 损失函数

### 特征提取网络

骨干网络采用C3D网络，输入为一系列RGB的视频帧，输入维度为$\mathbb{R} ^ {3 \times L \times H \times W}$，经过C3D网络的5层卷积网络，输出的特征图维度为$C_{conv5b} \in \mathbb{R} ^{512 \times \frac{L}{8} \times \frac{H}{16} \times \frac{W}{16}}$。此处的输出作为Proposal subnet和Classification subnet的公共注入。$H$和$W$为112，长度$L$试内存情况而定。

### 候选框子网络Temporal Proposal Subnet

为了检测出所有长度的proposal，在候选框子网络中引入faster R-CNN中的RPN网络。

Temporal Proposal Subnet输入为C3D特征，为$C_{conv5b} \in \mathbb{R} ^{512 \times \frac{L}{8} \times \frac{H}{16} \times \frac{W}{16}}$，每个时序上的特征被保存在512维的向量里，对于这个512维的向量，基于anchor segments计算时序偏差。

#### anchor segments生成

将anchor视频段分类为该anchor视频段是否有activity。anchor视频段初始化为多尺度视频段，均匀分布于以$\frac{L}{8}$为中心的视频段中，每个时间段有$K$个anchor，所以一个$L$长度的视频流共有$\frac{L}{8} * K$个anchor segments。

为了得到anchor segments在每个时间位置的的特征，在$C_{conv5b}$后增加一个$3*3*3$的3D卷积来扩展Temporal Proposal Subnet的感受野；然后将其从$\frac{H}{16} \times \frac{W}{16}$下采样到$1 \times 1$，生成一个仅针对时间的特征图$C_{tnp} \in \mathbb{R} ^{512 \times \frac{L}{8} \times1 \times 1}$。

