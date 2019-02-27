---
title: 表情识别项目梳理
date: 2019-02-26 16:09:55
tags:
- 计算机视觉
- 实习项目
- 表情识别
categories: Work
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# 表情识别项目

这个项目是去年8月份去海尔优家实习负责的项目，整个项目基本都自己负责，每周汇报推进，因此是在摸索中前进，没有具体的参考项目，代码实现放在我的[github项目](https://github.com/niuyuanyuanna/EmotionRecognition.git)

## 目标任务

搭建模型，识别Anger、Disgust、Fear、Happy、Sadness、Surprise、Neural七种表情数据。

### 数据集

#### CK+数据集

该数据库包括123个目标, 593个图像序列，每个图像序列的最后一帧图片都有action units 的标签，而在这593个图像序列中，有327个序列有表情的标签。

#### fer2013数据集

该数据库共包含35887张人脸图片，其中训练集28709张、验证集3589张、测试集3589张。数据库中的图片均为灰度图片，大小为48*48像素，样本被分为0=anger(生气)、1=disgust(厌恶)、2=fear(恐惧)、3=happy(开心)、4=sad(伤心)、5=surprised(惊讶)、6=normal(中性)七类，各种类型分布基本均匀。

#### RAF数据集

包括29672张图像，共七种表情，直接使用已经进行人脸对齐后的图像共有15,339个文件进行训练，每张图像为100*100像素大小。

#### AffectNet数据集

包含从网上收集的100多万张面部图像，是表情识别最大的数据集，其中45万张图片手工标注为7种表情。包含Neutral, Happy, Sad, Surprise, Fear, Anger,Disgust, Contempt, None, Uncertain, Non-face共11个类。

### 数据预处理

#### CK+数据集

选择CK+数据集中有表情标签的序列，提取其中表情强烈的图片制作数据集，并增强图片，增强方式包括随机裁剪，水平翻转，图片旋转，色彩调整（亮度、对比度、色相、饱和度等）。在本实验中采用随机裁剪、水平翻转以及亮度调整。由于数据集中图片为灰度图像，所以无法调整图像对比度、饱和度和色相。剪裁好的图像尺寸为200*200，得到数据集总数为12387张。

#### RAF数据集

下载好的RAF数据集中，使用align文件夹中的数据，对数据进行随机变换（随机变换包括水平翻转、调整图像饱和度、对比度、亮度等），然后对变换后的图像做z-score标准化，计算出数据集每个通道数的均值和方差，对所有像素点按照通道数不同减去均值除以方差，使得到的数据符合正态分布。公式为：
$$
x^* = \frac{x - \mu}{\sigma}
$$
其中$\mu$表示所有样本数据的均值，$\sigma$表示所有样本的标准差

此时所有图像均被映射为均值为0，方差为1的正态分布，这样的好处是：

- 把不同图片映射到同一坐标系，具有相同的尺寸；

- 像素大小不同的问题就可以转换为RGB分量具有相似特征分布的问题；

- 一定程度消除了因为图像亮度、质量不佳、有噪声等原因对模型权值的影响，加速收敛。

#### AffectNet数据集

该数据集有100w张，清洗后剩下20w+数据集可用，在train_list中每一类随机选择5000个数据，因此总数据个数为3.5w。

### 网络模型

#### Inception-V3

使用Inception-V3进行迁移学习，CK+数据集通过训练好的神经网络直接到达瓶颈层进行图像特征提取，替换最后一层全连接层，训练最后一层参数。在CK+数据集上其acc可达98.6%

#### 论文复现（AlexNet为基础）

根据A Real-time Facial Expression Recognizer using Deep Neural Network.使用fer2013数据集，自己搭建以AlexNet为基础的模型，减少3个卷积层。复现论文结果，在fer2013数据集上达到70.2%准确率。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/work/CNNModel.png" width=50%/>
</center>


#### 论文复现

Real-time CNN for Emotion and Gender Classification，采用了ResNet的网络结构，增加shortcut，网络结构为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/work/Shortcut.jpg" width=50%/>
</center>
在RAF数据集中训练，acc最大可达75%

#### MobileNets

MobileNets是为移动和嵌入式设备提出的高效模型。MobileNets基于流线型架构(streamlined)，使用深度可分离卷积(depthwise separable convolutions，即Xception变体结构)来构建轻量级深度神经网络。
深度可分离卷积的作用：把标准卷积分解成深度卷积(depthwise convolution)和逐点卷积(pointwise convolution)。这么做的好处是可以大幅度降低参数量和计算量。详见另一篇博客[MobileNets]()

网络结构为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/work/MobileNetModel.png" width=50%/>
</center>

在RAF训练集中训练，acc可达72.8%

#### DAN

结合人脸关键点检测进行表情分类，整体沿用DAN的方法，使用AffectNet作为训练集。
















