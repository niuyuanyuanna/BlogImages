---
title: 常见激活函数
date: 2018-07-22 13:47:54
tags:
- 神经网络
- 激活函数
categories: Deep Learning
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/deep_learning.jpg
---

# <center>神经网络常用激活函数</center>

## 为什么需要激活函数
激活函数的性质：

- **非线性**：激活函数为线性函数时，两层神经网络就可以拟合所有的线性函数。若激活函数为恒等激活函数，即$f(x)=x$时，不满足条件
- **可微性**：当优化方法是基于梯度的时候，必须满足可微性
- **单调性**：激活函数是单调函数时，单层网络才能保证是凸函数
- **$f(x)\approx x $**：当激活函数满足这个性质的时候，如果参数的初始化是random的很小的值，那么神经网络的训练将会很高效；如果不满足这个性质，那么就需要很用心的去设置初始值。
- **输出值的范围**：激活函数输出值的范围是有限的时候，基于梯度的优化方法更加稳定，因为特征的表示受有限权值的影响更显著；激活函数范围无限时，模型训练更高效，需要更小的learning rate。


## 常见的激活函数

### Sigmoid函数
Sigmoid又叫作 Logistic 激活函数，它将实数值压缩进 0 到 1 的区间内，还可以在预测概率的输出层中使用。该函数将大的负数转换成 0，将大的正数转换成 1。数学公式为：
$$
\sigma(x) = \frac{1}{1+e^{-x}}
$$

它是关于中心点（0, 0.5）对称的函数，图形为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/37147146.jpg" alt="sigmoid function" title="Sigmoid function" width=70%/>
</center>

Sigmoid函数导数为：
$$
\sigma{}'(x)=\sigma (x)(1-\sigma (x))
$$
图形为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/99194005.jpg" alt="sigmoid导数" title="Sigmoid function 导数" width=75%/>
</center>

#### 缺点：

- 会有梯度弥散。Sigmoids saturate and kill gradients. sigmoid 有一个非常致命的缺点，当输入非常大或者非常小的时候（saturation），这些神经元的梯度是接近于0的，从图中可以看出梯度的趋势。需要尤其注意参数的初始值来尽量避免saturation的情况。如果初始值很大，大部分神经元可能都会处在saturation的状态而把gradient kill掉，这会导致网络变的很难学习。
- 不关于原点对称。 output 不是0均值。这是不可取的，因为这会导致后一层的神经元将得到上一层输出的非0均值的信号作为输入。  产生的一个结果就是：如果数据进入神经元的时候是正的(e.g. $x>0$，elementwise in$f=w^{T}x + b$ )，那么$w$计算出的梯度也会始终都是正的。 如果输入神经元的数据总是正数，那么关于w的梯度在反向传播的过程中，将会要么全部是正数，要么全部是负数，这将会导致梯度下降权重更新时出现z字型的下降。
- 计算指数耗时。

### Tanh函数
tanh和sigmoid函数相似，实际上，tanh 是sigmoid的变形：

$$
tanh(x) = 2sigmoid(2x) -1
$$

与 sigmoid 不同的是，tanh 是0均值的。它的图像关于原点中心对称。因此，实际应用中，tanh 会比 sigmoid 更好。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/91091273.jpg" alt="tanh function" title="tant function" width=70%/>
</center>

#### 优点：

- 解决原点对称的问题。tanh解决了Sigmoid的输出是不是零中心的问题。
- 收敛速度比sigmoid更快

#### 缺点：

- 仍然没有解决梯度弥散问题

### ReLu函数系列：
#### ReLu

ReLU非线性函数图像如下图所示。相较于sigmoid和tanh函数，ReLU对于随机梯度下降的收敛有巨大的加速作用；sigmoid和tanh在求导时含有指数运算，而ReLU求导几乎不存在任何计算量。
其公式为：

$$
f(x) = max(0,x)
$$

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/48089832.jpg" alt="ReLu function" title="ReLu function" width=70%/>
</center>

##### 优点：

- 单侧抑制； 
- 相对宽阔的兴奋边界； 
- 稀疏激活性。 

##### 缺点：
- ReLU单元比较脆弱并且可能“死掉”，而且是不可逆的，因此导致了数据多样化的丢失。通过合理设置学习率，会降低神经元“死掉”的概率。 


#### Leakly ReLu
Leaky ReLUs用来解决 “dying ReLU” 问题。与 ReLU 不同的是：

$$
f(x)=
\begin{cases} 
\alpha x &  (x<0)\\ 
 x & (x>=0) 
\end{cases}
$$

此处$\alpha$是一个较小的常数，这样既修正了数据分布，又保留了负半轴的值，使得负半轴的信息不会全部丢失。
其图像为：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/50011864.jpg" alt="Leakly ReLu function" title="Leakly ReLu function" width=50%/>
</center>

#### Parametric ReLU(P-ReLu)
对于 Leaky ReLU 中的$\alpha$，通常都是通过先验知识人工赋值的。 然而可以观察到，损失函数对α的导数我们是可以求得的，可以将它作为一个参数进行训练，而且效果更好。
对$\alpha $的导数如下：

$$
\frac{\delta y_{i}}{\delta \alpha} = \left\{\begin{matrix}
0 & y_{i} > 0 \\ 
 y_{i}& other 
\end{matrix}\right.
$$

#### Randomized ReLU (R-ReLu)
Randomized Leaky ReLU是 leaky ReLU 的random 版本 （$\alpha$ 是随机的）。核心思想就是，在训练过程中，$\alpha$ 是一个服从高斯分布的 $U(l,u)$ 中 随机抽取出来的，然后在测试过程中进行修正（有点像dropout的用法）。
数学表达式为：

$$
y_{j,i}=\left\{\begin{matrix}
x_{j,i} &x_{j,i}\geq 0 \\ 
a_{j,i}x_{j,i} & x_{j,i}<0
\end{matrix}\right.
$$

其中$a_{j,i}\sim U(l,u)$，$l<u$且$u\in [0,1)$。在测试阶段，把训练过程中所有的$\alpha_{j,i}$取个平均值 。

测试阶段激活函数为：

$$
y_{ij}=\frac{x_{ij}}{\frac {l+u}{2}} 
$$

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/21922024.jpg" alt="P-ReLu and R-ReLu" title="P-ReLu and R-ReLu" width=100%/>
</center>

### Maxout
Maxout是对ReLU和leaky ReLU的一般化归纳，假设$w$是二维的，函数公式为：

$$
f(x) = max(w_{1}^{T}x+b_{1},w_{2}^{T}x+b_{2})
$$



Maxout的拟合能力是非常强的，它可以拟合任意的的凸函数。作者从数学的角度上也证明了这个结论，即只需2个maxout节点就可以拟合任意的凸函数了（相减），前提是”隐隐含层”节点的个数可以任意多。

Maxout非线性函数图像如下图所示。Maxout具有ReLU的优点，如计算简单，不会 saturation，同时又没有ReLU的一些缺点，如容易死掉。 

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/43488930.jpg" alt="Maxout" title="Maxout" width=100%/>
</center>

由公式可以看出每个神经元的参数都变为原来的两倍，导致整体参数增加。

### Softmax
Softmax用于多分类神经网络输出，目的是让大的更大。函数公式是：

$$
\sigma(z)_{j}=\frac{e^{z_{j}}}{ \sum_{k=1}^{K} e^{z_{k}}}
$$

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/deepLearning/90715034.jpg" alt="Softmax" title="Softmax" width=75% />
</center>

Softmax是Sigmoid的扩展，当类别数k＝2时，Softmax回归退化为Logistic回归。  