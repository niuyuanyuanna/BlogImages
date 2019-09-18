---
title: 数字图像基础
date: 2018-07-12 16:32:49
tags:
- 图像处理
- 原理
- CV
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# 数字图像处理

- 图像是一个二维亮度函数$f(x,y)$,其中$(x,y)$定义为空间坐标，则$f(x,y)$定义该点的亮度或灰度。
- 数字图像是指图像$f(x,y)$在空间坐标和亮度的数字化，数字图现象是由有限的元素组成，每一个元素都有一个特定的位置和幅值，这些元素称为图像元素或像素。

## 数字图像基础

用二维函数$f(x,y)$表示图像，在特定坐标$(x,y)$处，$f$的值是一个正标量，其物理意义由图像源决定。当一副图像从屋里过程产生时，他的值正比于物理源的辐射能量，则$f(x,y)$一定是非0且有限的，即：
$$
f(x,y) = i(x,y)r(x,y) \\
0 < i(x,y) < \infty \quad 0 < r(x,y) < \infty
$$
函数$f(x,y)$可以由两个分量表示：

1. 入射到观察场景的光源总和$i(x,y)$
2. 场景中物体反射光总量$r(x,y)$

### 图像取样和量化

一副连续的图像其$x$和$y$的坐标及幅值可能都是连续的，为了转换为数字图像，需要在坐标和幅度上做数字化操作。数字化坐标值称为取样，数字化幅度值称为量化。与数字信号处理中操作一致。

取样：一个二维取样函数为：
$$
S(x,y) = \sum_{m = - \infty }^{+ \infty} \sum_{n = - \infty }^{+ \infty} \delta (x - m \Delta x, y - n \Delta y) \\

\delta (x,y) = \left\{\begin{matrix}
1 & x=y=0\\ 
0 & others
\end{matrix}\right.
$$
取样后的图像为：
$$
\begin{equation}
\begin{split}
f_s(x,y) &= S(x,y)f(x,y) \\
&= \sum_{m = - \infty }^{+ \infty} \sum_{n = - \infty }^{+ \infty} f(x,y) \delta (x - m \Delta x, y - n \Delta y) \\
&= \sum_{m = - \infty }^{+ \infty} \sum_{n = - \infty }^{+ \infty} f(m\Delta x,n \Delta y) \delta (x - m \Delta x, y - n \Delta y)
\end{split}
\end{equation}
$$
取样和量化的结果是一个实际矩阵，可以用矩阵形式表示一副数字图像，也可以用变量和幅值都是整数的二维函数表示。

数字化过程对$M$和$N$除了必须取正整数外没有其他要求，灰度级典型的取值是2的整数次幂。对于一副大小为$M \times N$，灰度级$L = 2^k$的数字图像，所需的存储空间为$b =  M \times N \times k$。称该图像为k比特图像。

### 空间和灰度级分辨率

取样值是决定一副图像空间分辨率的主要参数。基本上，空间分辨率是图像中可辨别的最小细节。

广泛使用的分辨率的意义为每单位距离可分辨的线对数目，线对主要是为了说明反应原始场景中细节的能力。灰度级分辨率是指在灰度级别中可分辨的最小变化，当没有必要对设计像素的物理分辨率进行实际度量和在原始场景中分析细节等级时，通常把大小为$M \times N$，灰度为$L$级的数字图像称为空间分辨率为$M \times N$像素、灰度级为$L$的数字图像。

**改变取样数目对图像的影响**

最主观的效果就是图像大小发生改变，下图显示了一副$1024 \times 1024$像素的图像，其灰度级为8比特，其他图现象是对原图抽样的结果。

<div align=center>    
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/down-sample.png" width="80%">
</div>


采样可以通过删除行和删除列来进行，恢复可以通过复制行和复制列来进行。将抽样后的图像恢复为原来的大小，结果为：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/up-sample.png" width="80%">
</div>

**灰度级变化的影响**

保持取样数恒定，以2的整数次幂将灰度级从256减少至2.

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/grad-dowm-sample.png" width="80%">
</div>

**放大和收缩图像**

这种操作与取样量化图像之间的关键区别在于放大和收缩适用于数字图像。

要求执行两步操作：

1. 计算新的像素在原图的对应位置
2. 为这些对应位置赋值f

第一步，$f(x,y)$表示输出图像， $g(u,v)$为输入图像，进行几何运算，可定义为:

$$
f(x,y) = g(u_0, v_0) = g[a(x,y), b(x,y)] \\
u_0 = a(x,y) \quad v_0 = b(x,y)
$$

- 当$u_0 = x, v_0 = y$时，相当于直接将g拷贝到f，没有增加任何改动
- 当$u_0 = x + x_0, v_0 = y + y_0$时，表示当前图像被平移，点$(x_0, y_0)$被平移到了原点
- 当$u_0 = \frac {x}{c} , v_0 =\frac {y}{d}$时，图像在x轴方向放大c倍，y轴方向放大d倍。例：将一副$200 \times 200$的图像$g(u,v)$放大1.5倍，得到$300 \times 300$的新图$f(x,y)$。产生新图的过程实际就是为$300 \times 300$像素赋值的过程。如$f(100, 100) = g(\frac{100}{1.5}, \frac{100}{1.5}) = g(66.7, 66.7)$此时无法直接找到66.7，赋值的时候为第二步操作。

第二步，当f刚好可以找到g中对应的点时，直接赋值，当找不到时，需要进行插值。

常见插值操作：

- 最近邻法：取点$(u_0, v_0)$最近的整数坐标$(u,v)$
- 双线性插值：根据四个相邻点的灰度值通过插值计算
- 更多临点的内插

一维线性插值：

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/one-demention-insert.png" width="40%">
</div>

已知$(x_0, y_0)$与$(x_1,y_1)$的值，要得到这两点间曲线上$(x,y)$的值，可以得到：
$$
\frac{y - y_0}{y_1-y_0} = \frac{x - x_0}{x_1-x_0}
$$
令$\alpha = \frac{x - x_0}{x_1-x_0}$则$y = (1 - \alpha) y_0 + \alpha y_1$

<div align=center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/two-demention-insert.png" width="60%">
</div>

已知红点的四个点的坐标及幅值，需要找到绿点的幅值，根据四个红点进行两次一维线性插值找到两个蓝点的幅值，根据两个蓝点再进行一维线性插值，得到中间点的幅值。

### 像素间基本关系

#### 相邻像素

对于坐标$(x,y)$的一个像素p有4个水平和垂直的相邻像素，分别为$(x +1,y), (x - 1, y), (x, y +1), (x, y - 1)$，这个像素集合、称为p的4邻域，记为$N_4(p)$。p的4个对角邻像素$(x+1, y+1), (x-1,y-1),(x+1,y-1),(x-1,y+1)$称为p的D邻域，记为$N_D(p)$。4邻域和D邻域合在一起称为p的8邻域，记为$N_8(p)$。

#### 邻接性、连通性、区域、边界

令V是用于定义邻接性的灰度值集合，可考虑三种类型的邻接性。

- 4邻接：如果q在$N_4(p)$集中，具有V中数值的两个像素p和q是4邻接；
- 8邻接：如果q在$N_8(p)$集中，具有V中数值的两个像素p和q是8邻接；
- m邻接（混合邻接）下面两种情况满足一种即可
  - q在$N_4(p)$集中；
  - q在$N_D(p)$集中，且集合$N_4(p) \cap N_4(q)$中没有V值的像素，则具有V值的像素p和q是m邻接

如下图所示：设$V=\{ 1 \}$，点p和哪些点是4邻接的，和哪些点是8邻接的。

```
| 0(a) | 1(b) | 1(c) |
| :--: | :--: | :--: |
| 1(d) | 1(p) | 0(e) |
| 1(f) | 0(g) | 1(h) |
```

从图中可看出p和点b、d是4邻接，点b、c、d、f、h为8邻接。b、d、h是p的m邻接。

混合邻接是8邻接的改进，主要是为了消除采用8邻接发生的二义性。

<div align=center>
<img src="/Users/liuyuan/BlogImages/computerVersion/8linjie.jpg" width="60%">
</div>



