---
title: 数字图像处理基本原理及方法
date: 2018-07-12 16:32:49
tags:
- 图像处理
- 原理
- CV
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# 数字图像处理基本原理及方法

数字图像处理是指将图像信号转换成数字信号并利用计算机对其进行处理的过程。图像处理最早出现于 20 世纪 50 年代，当时的电子计算机已经发展到一定水平，人们开始利用计算机来处理图形和图像信息。数字图像处理作为一门学科大约形成于 20 世纪 60 年代初期。早期的图像处理的目的是改善图像的质量，它以人为对象，以改善人的视觉效果为目的。图像处理中，输入的是质量低的图像，输出的是改善质量后的图像，常用的图像处理方法有图像增强、复原、编码、压缩等。 

## 数字图像处理常用方法

1. 图像变换

由于图像阵列很大，直接在空间域中进行处理，涉及计算量很大。因此，往往采用各种图像变换的方法，如傅立叶变换、沃尔什变换、离散余弦变换等间接处理技术，将空间域的处理转换为变换域处理，不仅可减少计算量，而且可获得更有效的处理（如傅立叶变换可在频域中进行数字滤波处理）。目前新兴研究的小波变换在时域和频域中都具有良好的局部化特性，它在图像处理中也有着广泛而有效的应用。 

2. 图像编码压缩

图像编码压缩技术可减少描述图像的数据量（即比特数），以便节省图像传输、处理时间和减少所占用的存储器容量。压缩可以在不失真的前提下获得，也可以在允许的失真条件下进行。编码是压缩技术中最重要的方法，它在图像处理技术中是发展最早且比较成熟的技术。 

3. 图像增强和复原

图像增强和复原的目的是为了提高图像的质量，如去除噪声，提高图像的清晰度等。图像增强不考虑图像降质的原因，突出图像中所感兴趣的部分。如强化图像高频分量，可使图像中物体轮廓清晰，细节明显；如强化低频分量可减少图像中噪声影响。图像复原要求对图像降质的原因有一定的了解，一般讲应根据降质过程建立“降质模型”，再采用某种滤波方法，恢复或重建原来的图像。 

4. 图像分割

图像分割是数字图像处理中的关键技术之一。图像分割是将图像中有意义的特征部分提取出来，其有意义的特征有图像中的边缘、区域等，这是进一步进行图像识别、分析和理解的基础。虽然目前已研究出不少边缘提取、区域分割的方法，但还没有一种普遍适用于各种图像的有效方法。因此，对图像分割的研究还在不断深入之中，是目前图像处理中研究的热点之一。 

5. 图像描述

图像描述是图像识别和理解的必要前提。作为最简单的二值图像可采用其几何特性描述物体的特性，一般图像的描述方法采用二维形状描述，它有边界描述和区域描述两类方法。对于特殊的纹理图像可采用二维纹理特征描述。随着图像处理研究的深入发展，已经开始进行三维物体描述的研究，提出了体积描述、表面描述、广义圆柱体描述等方法。 

6. 图像分类

图像分类（识别）属于模式识别的范畴，其主要内容是图像经过某些预处理（增强、复原、压缩）后，进行图像分割和特征提取，从而进行判决分类。图像分类常采用经典的模式识别方法，有统计模式分类和句法（结构）模式分类，近年来新发展起来的模糊模式识别和人工神经网络模式分类在图像识别中也越来越受到重视。 

### 图像基本属性
1. 亮度（brightness）也称为灰度，它是颜色的明暗变化，常用 0 ％～ 100 ％ ( 由黑到白 ) 表示。
2. 色调（Hue）
3. 饱和度（Saturation）
4. Hue + Saturation = chromaticity色度
5. 直方图（histogram）表示图像中具有每种灰度级的象素的个数，反映图像中每种灰度出现的频率。图像在计算机中的存储形式，就像是有很多点组成一个矩阵，这些点按照行列整齐排列，每个点上的值就是图像的灰度值，直方图就是每种灰度在这个点矩阵中出现的次数。
6. 对比度（contrast）

#### 数字图像间的距离

- 欧式距离（Euclidean）：$D_{E}[(i,j),(h,k)] = \sqrt {(i-h)^{2} + (j - k)^{2}}$
- 城市距离（City block）：$D_{4}[(i,j),(h,k)] =\left | i-h\right | + \left | j - k\right |$
- 棋盘距离（Chessboard）：$D_{8}[(i,j),(h,k)]=max ( |i-h|+|j-k| )$



#### 图像增强相关操作
1. 直方图均衡（Histogram equalization ）
2. 直方图匹配（Histogram matching ）
3. 局部增强（Local enhancement）

#### 常见噪声

- 加性噪声：噪声和图像信息独立$f(x,y) = g(x, y) + v(x,y)$
- 乘性噪声：噪声量级取决于图像信息的量级$f(x,y) =  g(x, y) +  g(x, y) \times v(x,y)$
- 量化噪声：当量化强度不够时会产生量化噪声
- 冲击噪声：典型代表椒盐噪声（salt and pepper noise）用中值滤波去除


### 图像预处理

图像预处理分为四个类，分别为：

1. pixel brightness transformations 像素亮度变换
2. geometric transformation几何变换
3. pre-processing methods that use a local neighborhood of the processed pixel 使用处理过的像素处理未处理过的像素
4. image restoration 图像重建

#### pixel brightness transformations 像素亮度变换
pixel brightness transformations 像素亮度变换，其变化只取决于像素本身，其包括两种：
   - Brightness Corrections：仅考虑图像中原来的亮度及像素位置；
   - Gray Scale Transformations（灰度变换） ：改变像素亮度时和像素在图中的位置无关。对于输入图象$f(x，y)$，输出图像$g(x，y)$，$T(in)$为灰度变换函数，则$g(x，y)=T( f(x，y) )$。
        - 主要目的：
             - 改善画质，使图像显示效果更加清晰
             - 有选择性地突出图像中感兴趣的特征或抑制某些不需要的特征，使图像与视觉响应特征相匹配（图像增强）
        - 主要应用：
             - 图像求反：这种方法适用于增强嵌入图像暗色区域的白色或灰色细节。
             - 对比度拉伸；
             - 图像灰度分割（二值化）：在图像处理领域，二值图像运算量小，并且能够体现图像的关键特征，因此被广泛使用。

### 图像增强

#### 直方图均衡
通过灰度变换将一幅图像转换为另一幅具有均衡直方图的图像，即在一定灰度范围内具有相同的象素点数的图像的过程。通过这种方法，亮度可以更好地在直方图上分布。这样就可以用于增强局部的对比度而不影响整体的对比度，直方图均衡化通过有效地扩展常用的亮度来实现这种功能。
这种方法对于背景和前景都太亮或者太暗的图像非常有用，这种方法尤其是可以带来X光图像中更好的骨骼结构显示以及曝光过度或者曝光不足照片中更好的细节。这种方法的一个主要优势是它是一个相当直观的技术并且是可逆操作，如果已知均衡化函数，那么就可以恢复原始的直方图，并且计算量也不大。
这种方法的一个缺点是它对处理的数据不加选择，它可能会增加背景噪声的对比度并且降低有用信号的对比度。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/57057553.jpg" width="90%">
</div>

#### 直方图匹配
直方图规定化（histogram specification）又称直方图匹配，是指使一幅图像的直方图变成规定形状的直方图而对图像进行变换的增强方法。就是通过一个灰度映像函数，将原灰度直方图改造成所希望的直方图。所以，直方图修正的关键就是灰度映像函数。
直方图规定化原理是对两个直方图都做均衡化，变成相同的归一化的均匀直方图。以此均匀直方图起到媒介作用，再对参考图像做均衡化的逆运算即可。直方图均衡化是直方图规定化的桥梁。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/90622666.jpg" width="90%">
</div>

#### 局部增强
Local preprocessiong 也称为filtering，根据图像处理的目的分为两类：
#####  平滑（smoothing）
使用低通滤波器可以抑制高频成分（sharp image details），使图像变模糊。图像平滑的目的是抑制噪声及图片中的波动，和在频域抑制高频的作用相同。
缺点：平滑会损失边缘信息，丢失重要的边界信息。filter尺寸越大，丢失信息越多。

- Image Smoothing-Average**均值滤波**
  均值滤波是一种**线性滤波**操作，输出图像的每一个像素是核窗口内输入图像对应像素的像素的平均值( 所有像素加权系数相等)。相当于给图像经过一个$n*n$的卷积核，卷积核的系数都为1。
  均值滤波算法比较简单，计算速度快，但是均值滤波本身存在着固有的缺陷，即它不能很好地保护图像细节，在图像去噪的同时，也破坏了图像的细节部分，从而使图像变得模糊，不能很好地去除噪声点。但均值滤波对周期性的干扰噪声有很好的抑制作用。

- Median Filter **中值滤波**
  中值滤波法是一种**非线性平滑技术**，将图像的每个像素用邻域 (以当前像素为中心的正方形区域)像素的**中值**代替 ，常用于消除图像中的**椒盐噪声**。

  与低通滤波不同的是，中值滤波对脉冲噪声有良好的滤除作用，特别是在滤除噪声的同时，**能够保护信号的边缘，使之不被模糊**，但它会洗去均匀介质区域中的纹理。这些优良特性是线性滤波方法所不具有的。
  中值滤波能减弱或消除傅里叶空间的高频分量，同时也影响低频分量。中值滤波去除噪声的效果依赖于两个要素：邻域的空间范围和中值计算中涉及的像素数。一般说来，小于滤波器面积一半的亮或暗的物体基本上会被滤除，而较大的物体几乎会原封不动地保存下来，因此中值滤波器的空间尺寸必须根据现有的问题来进行调整。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/25880713.jpg" width="90%">
</div>

- Maximum and Minimum Filters极大极小滤波器
极大值极小值滤波器是两个串联滤波器，可以用于消除椒盐噪声。极大值滤波器移除pepper-type（小值）噪声，极小值滤波器移除salt-type（大值）噪声。

##### 锐化（sharpening）
使用高通滤波器可以抑制低频成分，锐化图像的边缘，增强图片细节

- Derivative operators **差分滤波器**
一阶微分：$\frac{\partial f}{\partial x}= f(x+1) - f(x)$
一阶微分会产生较粗的边缘，因为沿着斜坡的积分非零。主要检测极值。
二阶微分：$\frac{\partial ^{2} f}{\partial x^{2}}= f(x+1) + f(x-1) -2f(x)$
二阶微分在增强细节方面要比一阶微分好得多，这是一个适合锐化图像的理想特性。因此常用二阶微分来进行图像增强。典型的二阶微分算子是**拉普拉斯算子**：
$$
\triangledown^{2}f=\frac{\partial ^{2} f}{\partial x^{2}}+\frac{\partial ^{2} f}{\partial y^{2}}
$$
$$
\frac{\partial ^{2} f}{\partial x^{2}}= f(x+1,y) + f(x-1,y) -2f(x,y)
$$
$$
\frac{\partial ^{2} f}{\partial x^{2}}= f(x,y+1) + f(x,y-1) -2f(x,y)
$$
$$
\triangledown^{2}f= f(x+1,y) + f(x-1,y) + f(x,y+1) + f(x,y-1)-4f(x,y)
$$

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/20812166.jpg" width="90%">
</div>

由于拉普拉斯是一种微分算子，因此其应用强调的是图像中的灰度突变，并不强调灰度级缓慢变化的区域。
将原图像和拉普拉斯图像叠加在一起的简单方法，可以复原背景特性并保持拉普拉斯锐化处理的效果。
$$
g(x,y) = f(x,y) + \triangledown^{2}f(x, y)
$$
如果所使用的模板定义有负的中心系数，那么必须将原图像减去经拉普拉斯变换后的图像，而不是加上他，从而得到锐化后的结果。

- High-boost Filter**高频提升滤波器**

高通滤波后的图像= 原图-低通滤波后的图像
$$
\begin{equation}
\begin{aligned}
Hight-boost & = A \times original - lowpass(blurred) \\
& = A \times original - lowpass(blurred) \\
& = (A-1) \times original + original - lowpass(blurred) 
 \end{aligned}
\end{equation}
$$
当$A=1$时，为标准的高通滤波器，

当$A > 1$时，部分原图抵消了高通滤波部分，保留了低频部分。

高频提升滤波器是图像处理的基础工具，常用于印刷出版业。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/2387564.jpg" width="90%">
</div>

带通滤波器移除选择的频带，用于图像重建，不用于图像增强

- 卷积和相关

卷积是一个filter的作用，相关是两个信号关联的度量。

#### 频域增强

##### 傅里叶变换（FT）

$$
\mathfrak{F}(f(x))= \int_{-\infty }^{+\infty}f(x)\cdot e^{-j2\pi ux}dx
$$

反变换：
$$
\mathfrak{F}  ^{-1} (F(u) ) = \int_{-\infty }^{+\infty}F(u)\cdot e^{j2\pi ux}du
$$
离散傅里叶变化可以分为实数部分和虚数部分：
$$
\begin{equation}
\begin{aligned}
F(u) &= \frac{1}{M}\sum_{x=0}^{M-1}f(x)[cos\frac{2\pi ux}{M} - jsin\frac{2\pi ux}{M}]\\
&=R(u)+jI(u)\\
&=\left | F(u) \right |e^{-j\varphi (u)}
 \end{aligned}
\end{equation}
$$
其中幅值和相位分别为：
$$
\left | F(u) \right | =\sqrt{ R^{2}(u)+I^{2}(u)}\\
\varphi (u)=arctan()\frac{I(u)}{R(u)}\\
P(u) = \left | F(u) \right |^{2} = R^{2}(u)+I^{2}(u)
$$
时域位移和频域位移的对应关系：
$$
f(x,y)e^{j2\pi (\frac{u_{0}x}{M} + \frac{v_{0}y}{N})}\Leftrightarrow F(u-u_{0}, v-v_{0})\\
f(x-x_{0}, y-y_{0}) \Leftrightarrow F(u,v)e^{-j2\pi (\frac{u_{0}x}{M} + \frac{v_{0}y}{N})}
$$
也即是在时域移动$(x_{0},y_{0})$，对应频域上是幅值不变，相位移动。

频域增强操作：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/34598712.jpg" width="90%">
</div>

频域滤波器分类：

- 低通滤波器：图像模糊、平滑——例如巴特沃斯滤波器、高斯低通滤波器
- 高通滤波器：增大高频幅值，对应锐化操作——巴特沃斯高通滤波器、高斯高通滤波器



###### Butterworth低通滤波器

$$
H(u,v)=\frac{1}{1+(\frac{D(u,v)}{D_{0}})^{2n}}
$$

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/55623921.jpg" width="90%">
</div>

- n越大，Butterworth低通滤波器越接近于理想低通滤波器，但会出现振铃效应
- 其有3db不变性
- 有平坦性

###### Gaussian低通滤波器

$$
H(u,v)=e^{\frac{-D^{2}(u,v)}{2\sigma ^{2}}}
$$

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/13819824.jpg" width="90%">
</div>

- $D_{0}$越小，越接近理想低通滤波器

低通滤波器应用：平滑印刷字体

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/90373906.jpg" width="90%">
</div>

###### Butterworth高通滤波器

$$
H_{hp}(u,v)=1-H_{lp}{(u,v)}\\
H(u,v)=\frac{1}{1+(\frac{D_{0}}{D(u,v)})^{2n}}
$$

和低通滤波器正好相反。增加一个常数项保存低频部分信息。

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/50235930.jpg" width="90%">
</div>

###### Gaussian高通滤波器

$$
H(u,v)=1-e^{\frac{-D^{2}(u,v)}{2\sigma ^{2}}}
$$

$\sigma = D_{0}$是cutoff频率

##### 频域拉普拉斯变换

时域微分对应傅里叶变换为频域乘上$(ju)$：
$$
F[\frac{d^{n}f(x)}{dx^{n}}] = (ju)^{n}F(u)\\
F[\frac{\partial ^{2}f(x,y)}{\partial x^{2}}+\frac{\partial ^{2}f(x,y)}{\partial y^{2}}] =- (u^{2}+v^{2})F(u,v)
$$
因此Laplacian可以看做是使用$H(u,v) = -(u^{2}+v^{2})$的滤波器。

带有中心点的Laplacian，其傅里叶变换对为：
$$
\triangledown ^{2}f(x,y) \Leftrightarrow -[(u-\frac{M}{2})^{2} +(v - \frac{N}{2})^{2}]F(u,v)
$$
在频域，Laplacian相当于一个函数中心在$(\frac{M}{2}, \frac{N}{2})$，且在顶点处的值为0，其他值为负的函数。

图像增强使用方法是原图减去拉普拉斯变换后的图：
$$
g(x,y) = f(x,y)-\triangledown^{2}f(x,y)\\
g(x,y) =\mathfrak{F} ^{-1} ( 1-[(u-\frac{M}{2})^{2} +(v-\frac{N}{2})^{2}]F(u,v) )
$$

###### Unsharp masking

$$
H_{hp}(u,v)=1-H_{lp}{(u,v)}\\
$$

###### High-boost filtering

$$
H_{hp}(u,v)=(A-1)+H_{hp}{(u,v)}\\
$$

###### Hight frequency emphasis filtering

$$
H_{hfe}(u,v)=a+bH_{hp}{(u,v)}
$$



#### 高斯金字塔

高斯金字塔背后的理论基础为尺度空间理论。 这个的概念可以用在任意维度的信号中，不过最常用的地方还是在二维的影像信号上，以二维影像信号作为主要讨论对象。 给定一张图片$ f(x,y)$它的尺度空间表示方式$ L(x,y;t)$定义为:影像信号$f(x,y)$和高斯函数$g(x,y;t)={\frac {1}{2{\pi }t}}e^{-(x^{2}+y^{2})/2t}$的卷积，完整公式为：
$$
L(x,y;t)=g(x,y;t)*f(x,y)
$$
式中的分号代表卷积的对象为 $x,y$而分号右边的$t$表示定义的尺度大小。 这个定义当$ t\geq 0$ 时对于所有的$t$都会成立，不过通常在实际操作时只会选取特定的$t$值。 其中$t$为高斯函数的变异数。当$t$趋近于零的时候，$g$成为一个单位脉冲响应，使得$ L(x,y;t)\ =f(x,y)$ ，这代表当 $t=0$的时候我们可以把这项操作视为图片$f$本身。 当$t$增加时，$L$代表将影像$f$通过一个较大的高斯滤波器，从而使得影像的细节被去除更多。

在建立高斯金字塔的时候，首先会将影像转换为尺度空间的表示方式，亦即乘上不同大小的高斯函数，之后再依据取定的尺度向下取样。 乘上的高斯函数大小和向下取样的频率通常会选为2的幂次，也就是说，在每次迭代的过程中，影像都会被乘上一个固定大小的高斯函数，并且被以长宽各0.5的比率被向下取样。 如果将向下取样过程的图片一张一张叠在一起，会呈现一个金字塔的样子，因此这个过程称为高斯金字塔。 

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/98042535.jpg" width="75%">
</div>

高斯金字塔使用：

- 如果物体的尺寸很小或者说对比度不高，通常则需要采用较高的分辨率来观察。
- 如果物体的尺寸很大或者说对比度很强，那么就仅仅需要较低的分辨率就能够来传观了。
- 那如果现在物体的尺寸有大有小，对比度有强有弱，这些关系同时存在，这个时候需要使用多分辨率处理



#### 同态滤波器（Homomorphic Filter）

对于一副图像$f(x,y)$可由照射分量$i(x,y)$和反射分量$r(x,y)$的乘积，即 
$$
f(x,y)=i(x,y)*r(x,y)
$$
由于照度相对变化很小，可以看作是图像的低频成份，而反射率则是高频成份。通过分别处理照度和反射率对像元灰度值的影响，达到揭示阴影区细节特征的目的。 

上式不能直接用于对照度和反射的频率分量进行操作，因此上式取对数  :
$$
Inf(x,y)=lni(x,y)+lnr(x,y)
$$
对上式两边取傅里叶变换：
$$
\mathfrak{F}(Inf(x,y) )=\mathfrak{F} (Ini(x,y) )+\mathfrak{F}  (Inr(x,y) )
$$
图像的照射分量通常由慢的空间变化来表征，而反射分量往往引起突变，特别是在不同物体的连接部分。这些特性导致图像取对数后的傅里叶变换的低频成分与照射相联系，而高频成分与反射相联系。  

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/672660.jpg" width="50%">
</div>

使用同态滤波器可以更好地控制照射分量和反射分量。这种控制器需要指定一个滤波器函数$H(u,v)$，它可用不同的可控方法影响傅里叶变换的低频和高频。如果$γL$和$γH$选定，而$γL<1$且$γH>1$，那么滤波器函数趋近于衰减低频（照射）的贡献，而增强高频反射的贡献。最终结果是同时进行动态范围抑制低频，增强对比度。  

使用同态滤波器步骤：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/62116892.jpg" width="50%">
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/56382828.jpg" width="50%">
</div>

同态滤波是一个比较经典的算法，有论文说可以去雾。但对水中图像效果确是极好的。另外同态滤波主要用于预处理阶段去除光照不均的影响，这用顶帽变化也可以的。 

### 图像重建

目标：

- 在某些方面提升图像
- 使用图像退化的先验知识来修复图像
- 主要方法即创建图像退化模型，反向操作处理退化的图像

重建包括两部分，一个是退化函数，另一部分是噪声。

在空间域中，生成模型形式为：
$$
d(x,y) = h(x,y) * I(x,y) + n(x,y)
$$
频域中，生成模型为：
$$
D(u,v) = H(u,v)I(u,v) + N(u,v)
$$
#### 造成图像退化的原因：

- 物体和摄像头的相对移动

  这时采用的$H(u,v)=\frac{sin(\pi VTu)}{\pi VTu}$，其中$T$是拍摄时间

- 不当的镜头焦距

  此时的$H(u,v) = \frac{j_{1}(ar)}{ar}$，其中$j_{1}$----

- 大气扰动

#### 逆滤波（Inverse filtration）
与噪声无关，限定半径
#### 维纳滤波（Wiener filtration）
在噪声环境下效果很好

  

  