---
title: Two-Stream Convolutional Networks for Action Recognition in Videos论文笔记
date: 2018-12-18 20:28:24
tags:
- 计算机视觉
- Action Recognition
- 论文笔记
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# Two-Stream Convolutional Networks for Action Recognition in Videos论文笔记

论文原文地址：[Two-Stream Convolutional Networks for Action Recognition in Videos](https://arxiv.org/abs/1406.2199)

这篇文章是NIPS 2014年提出一个two stream网络来做video action的分类，比较经典。two stream表示两个并行的网络：spatial stream convnet 和 temporal stream convnet. 这两个并行网络的作用是：空间域上从静态图像帧中识别动作；时域上使用密集光流特征识别动作。最后进行信息融合。

## 主要贡献

1. 提出了结合空域和时域网络的two_stream卷积网络结构。
2. 验证了即使在较小规模的训练数据集上，在多帧稠密光流上训练的卷积神经网络可以获得非常好的性能。
3. 展示了多任务学习，应用于不同的运动分类数据集，可以同时提升数据集的规模和检测性能。

## 相关工作

### 基于时空域特征的浅层高维编码

该方法是神经网络还未出现的传统视频处理方法。

#### HOG（Histogram of Oriented Gradient）

为梯度方向直方图，通过计算和统计图像局部区域的梯度直方图构成特征。因为其是在静态图像中提取的，所以为空间特征。
HOG用于目标检测。一种解决人体目标检测的图像描述子，是一种用于表征图像局部梯度方向和梯度强度分布特性的描述符

##### 步骤

1. 灰度化，标准化gamma空间和颜色空间；

gamma压缩公式：

$$
I_{(x,y)} = I_{(x,y)}^{gamma}
$$

2. 计算图像在x方向和y方向的梯度

常用的计算梯度的方法是使用Sobel算子和图像卷积，得到x方向和y方向的梯度：
$$
G_x = \begin{bmatrix}
-1 & 0 & 1
\end{bmatrix}\ast A \\
G_y = \begin{bmatrix}
-1 \\ 
0 \\ 
1
\end{bmatrix}\ast A 
$$
得到梯度特征的幅值和夹角
$$
G = \sqrt{G_x^2 + G_y^2} \\
\theta = arctan \frac{G_y}{G_x}
$$

3. 为每个cell构建梯度直方图
假设每个cell像素为$8 \ast 8$，切块后根据下面的星状图将对应夹角的幅值增加到对应的块中，形成一个$1 \ast 9$的特征向量。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/HOF.png" width="75%">
</div>

4. 将多个cell拼接为1个block，归一化得到block的特征向量

- 对每个cell特征向量归一化
- 拼接$n \ast n$个cell为一个block，特征向量此时变为$1 \ast n^2$
- 归一化1个block的特征向量，滑窗式移动到下一个block

5. 将所有block的特征vector拼接为特征matrix

##### 优点

1. 对图像几何的和光学的形变都能保持很好的不变性
2. 在粗的空域抽样、精细的方向抽样以及较强的局部光学归一化等条件下，只要行人大体上能够保持直立的姿 势，可以容许行人有一些细微的肢体动作，这些细微的动作可以被忽略而不影响检测效果。

#### HOF（Histogram of Flow）

HOF是基于光流法的直方图，因此需要了解光流法的计算。

##### Optical Flow

光流是由于场景中前景目标本身的移动、相机的运动，或者两者的共同运动所产生的。

- 其计算方法可以分为三类：

1. 基于区域或者基于特征的匹配方法
2. 基于频域的方法
3. 基于梯度的方法

简单来说，光流是空间运动物体在观测成像平面上的像素运动的“瞬时速度”。光流的研究是利用图像序列中的像素强度数据的时域变化和相关性来确定各自像素位置的“运动”。研究光流场的目的就是为了从图片序列中近似得到不能直接得到的运动场。

- 前提假设：

1. 相邻帧之间的亮度恒定
2. 相邻视频帧的取帧时间连续，或者，相邻帧之间物体的运动比较“微小”
3. 保持空间一致性；即，同一子图像的像素点具有相同的运动

运动场，其实就是物体在三维真实世界中的运动；光流场，是运动场在二维图像平面上的投影。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/OF.png" width="75%">
</div>

- 目的：

对于图片中的每个像素点找到其速度向量$\vec{u} = (u, v)$。

- 步骤：

1. 由于亮度恒定和微小运动，可以假设像素点灰度$I_{(x,y,t)} = I_{(x+dx, y+dy, t+dt)}$
2. 对上式进行Taylor一阶展开：

$$
I_{(x+dx, y+dy, t+dt)} = I_{(x,y,t)} + \frac{\partial I}{\partial x} dx + \frac{\partial I}{\partial y} dy + \frac{\partial I}{\partial t} dt
$$

得到：
$$
I_xdx + I_ydy + I_tdt = 0
$$
令$u = \frac{dx}{dt}$, $v = \frac{dy}{dt}$，则得到：
$$
I_xu + I_yv = -I_t
$$
即：
$$
\begin{bmatrix}
 I_x & I_y 
\end{bmatrix} \ast
\begin{bmatrix}
 u \\
 v 
\end{bmatrix}
=-I_t
$$
在一个小的邻域内，亮度恒定，则在这个邻域内的像素点满足：
$$
\begin{bmatrix}
 I_{x_1} & I_{y_1} \\
 I_{x_2} & I_{y_2} \\
 \vdots  &  \vdots
\end{bmatrix} \ast
\begin{bmatrix}
 u \\
 v 
\end{bmatrix} = -
\begin{bmatrix}
 I_{t_1}  \\
 I_{t_2} \\
 \vdots 
\end{bmatrix}
$$
即需要满足$A \vec{u} = b$，因此光流法主要是为了使得$\left \| A \vec{u} -b \right \|^2$有最小值。

3. 若$A \vec{u} = b$，则可以使用矩阵计算出$\vec{u} = (A^TA)^{-1} A^Tb$

- 光流法用于目标跟踪的原理：

1. 对一个连续的视频帧序列进行处理
2. 针对每一个视频序列，利用一定的目标检测方法，检测可能出现的前景目标
3. 如果某一帧出现了前景目标，找到其具有代表性的关键特征点（可以随机产生，也可以利用角点来做特征点）
4. 对之后的任意两个相邻视频帧而言，寻找上一帧中出现的关键特征点在当前帧中的最佳位置，从而得到前景目标在当前帧中的位置坐标
5. 如此迭代进行，便可实现目标的跟踪

##### 步骤

1. 计算每帧图像对应的光流场$\vec{u} =\begin{bmatrix} u \\ v \end{bmatrix} $
2. 计算光流矢量与横轴夹角及幅值：

$$
\theta = arctan \frac{v}{u} \\
U = \sqrt{u^2 + v^2}
$$

3. 使用同HOG相同的方法将夹角分为几个区域，绘制直方图，得到特征向量
4. 归一化特征向量

##### 补充

1. 以横轴为基准计算夹角能够使HOF特征对运动方向（向左和向右）不敏感
2. 通过归一化直方图实现HOF特征的尺度不变性
3. HOF直方图通过光流幅值加权得到，因此小的背景噪声对直方图的影响微乎其微
4. 通常直方图bin取30以上识别效果较好

HOF提取后的特征被编码为BOF（特征词袋）表示，并使用SVM进行线性分类

### 基于密集轨迹点

dense point trajectories ，这部分为主要为IDT，由调整局部描述符支持区域组成，可以跟随轨迹，通过光流计算。论文待整理。

### 基于神经网络

这些工作的大多数，网络的输入为堆叠的连续的视频帧，网络的输入为堆叠的连续的视频帧，所以模型被希望能够在第一层学习到时空域基于运动的特征。

## Two-Stream结构

网络结构由两部分组成，一个是空域上的，以单个的视频帧表象的形式存在，携带视频中的场景和目标信息；一个是时域上的，以视频帧间的运行形式存在，传递观察者(相机)和目标的移动。
其结构为：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/TwoStream.png" width="90%">
</div>


class score fusion考虑两种融合方案：一种是训练多分类的线性SVM，另一种为基于堆叠的L2-规范化的softmax分数

### 光流分支

输入为**一些连续视频帧的堆叠光流位移场**

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/OpticalFlow.png" width="90%">
</div>
1. (a)表示前一帧图像，(b)表示后一帧图像
2. (c)表示根据两帧图像计算出的Optical Flow
3. (d)(e)分别表示displacement vector field的水平和竖直两部分

#### 输入

在第$\tau$帧的时候输入即为$I_\tau$，论文中介绍了两种获取$I_\tau$的方法，一种是Optical Flow Stacking（光流栈），另一种是Trajectory Stacking（轨迹叠加）

- Optical Flow Stacking

$I_\tau(u, v, c)$表示$(u,v)$这个位置的像素点的displacement vector（位移矢量），$c$的取值范围为1~2L，其中L表示vedio的帧数，因为有横纵两个方向的vector，因此范围为2L。水平方向和竖直方向的$I_\tau(u, v, c)$计算公式为：

$$
I_\tau(u, v, 2k-1) = d_{\tau + k -1}^{x}(u,v) \\
I_\tau(u, v, 2k) =  d_{\tau + k -1}^{y}(u,v) \\
u = [1;w] \quad v=[1;h] \quad k=[1;L]
$$

这种方法是光流的简单叠加。简单的来说就是计算每两帧之间的光流，然后简单的stacking。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/OpticalFlowStacking.png" width="50%">
</div>

上面的图表示为

- Trajectory Stacking

轨迹叠加就是假设第一帧的某个像素点，我们可以通过光流来追踪它在视频中的轨迹。而简单的光流场叠加并没有追踪，每个都是计算的某帧$\tau + 1$中某个像素点P相对于$\tau$帧中对应像素点q的位移，光流场叠加最终得到的是每个像素点的两帧之间的光流图。
$$
I_\tau(u, v, 2k-1) = d_{\tau + k -1}^{x}(P_k) \\
I_\tau(u, v, 2k) =  d_{\tau + k -1}^{y}(P_k) \\
u = [1;w] \quad v=[1;h] \quad k=[1;L]
$$

其中$P_k$是轨迹中的第k个点，其计算方式为：
$$
P_1 = (u,v) \\
P_k = P_{k-1} + d_{\tau + k - 2}(p_{k-1})
$$

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/TrajectoryStacking.png" width="60%">
</div>
此时$I_\tau(u, v, c)$存储的值为该像素点的运动轨迹。

- 输入帧选择

上述两个方法其实考虑的都是前馈光流，我们都是依靠后一帧计算相对于前一帧的光流。当我们考虑T帧时，我们不再一直往后堆L帧，而是计算T帧之前L/2和T帧之后的L/2帧。

- 输入去噪

在输入光流前要减去平均光流，排除车相机的运动导致的负面影响。

- 最终输入

从$I_\tau$中采样为$224*224*2L$输入神经网络中。

### 多任务学习

spatial stream convnet因为输入是静态的图像，因此其预训练模型容易得到（一般采用在ImageNet数据集上的预训练模型），但是temporal stream convnet的预训练模型就需要在视频数据集上训练得到，但是目前能用的视频数据集规模还比较小（主要指的是UCF-101和HMDB-51这两个数据集，训练集数量分别是9.5K和3.7K个video）。因此作者采用multi-task的方式来解决。首先原来的网络（temporal stream convnet）在全连接层后只有一个softmax层，现在要变成两个softmax层，一个用来计算HDMB-51数据集的分类输出，另一个用来计算UCF-101数据集的分类输出，这就是两个task。这两条支路有各自的loss，最后回传loss的时候采用的是两条支路loss的和。



