---
title: R-CNN目标检测方法
date: 2018-07-26 17:35:20
tags:
- CV
- object detection
categories: Computer Version
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.jpeg
---
# 物体检测R-CNN介绍
## R-CNN
R-CNN的贡献主要是第一次将CNN引入到目标检测中，主要解决两个问题：

- 速度：经典的目标检测算法使用滑动窗法依次判断所有可能的区域。R-CNN则预先提取一系列较可能是物体的候选区域，之后仅在这些候选区域上提取特征，进行判断。
- 训练集：经典的目标检测算法在区域中提取人工设定的特征（Haar，HOG）。R-CNN则需要训练CNN进行特征提取。可供使用的有两个数据库：
	- 一个较大的识别库（ImageNet ILSVC 2012）：标定每张图片中物体的类别。一千万图像，1000类。 
	- 一个较小的检测库（PASCAL VOC 2007）：标定每张图片中，物体的类别和位置。一万图像，20类。

### 检测过程
R-CNN的检测过程主要分为四个步骤：

- 使用**Selective Search**方法，每张图片生成1K~2K个RoI；
- 对每个RoI使用**CNN**提取特征；
- 将feature vector 送入**SVM**二分类器中，判断是否属于该类；
- bounding-box regression精细修正RoI位置。

其整体识别流程如下图所示：
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/83769262.jpg" alt="R-CNN" title="R-CNN" width=75%/>

#### Selective Search

选择性搜索是一种用于目标检测的区域推荐算法。它的设计速度快，召回率高。它是根据颜色、纹理、大小和形状的兼容性，计算相似区域的层次分组。 

##### 候选区

- 使用一种分割手段，将图像分割成小区域 
- 查看现有小区域，合并可能性最高的两个区域。重复直到整张图像合并成一个区域位置 
- 输出所有曾经存在过的区域，所谓候选区域 
- 候选区域生成和后续步骤相对独立，实际可以使用任意算法进行。

##### 合并规则（优先合并以下四种区域）

- 颜色（颜色直方图）相近的 
- 纹理（梯度直方图）相近的 
- 合并后总面积小的 
- 合并后，总面积在其BBOX中所占比例大的

##### 多样化和后处理
为尽可能不遗漏候选区域，上述操作在多个颜色空间中同时进行（RGB,HSV,Lab等）。在一个颜色空间中，使用上述四条规则的不同组合进行合并。所有颜色空间与所有规则的全部结果，在去除重复后，都作为候选区域输出。

#### 特征提取

##### 预处理

将每个region都wrap到固定的大小（227×227） 。此处scale时需要注意：外扩的尺寸大小，形变时是否保持原比例，对框外区域直接截取还是补灰 。

##### CNN

将所有变形后的region输入到CNN中，网络结构为Hinton 2012年在Image Net上的分类网络AlexNet，其网络结构为：
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/66806389.jpg" alt="AlexNet" title="AlexNet" width=75% />

AlexNet使用了两块GPU训练神经网络。此网络提取的特征为4096维，之后送入一个4096->1000的全连接(fc)层进行分类。  学习率0.01。 
训练的过程分为两步：

- **pre-training** 使用ILSVRC 2012的数据集进行有监督的分类训练（迁移训练），初步训练出CNN每层的参数，输出1000维类别标号；
- **fine-tuning** 使用PASCAL VOC2007数据集，将第2步训练出来的CNN模型替换最后一层，变为N+1个输出，采用SGD方法训练最后一层参数。

##### 分类
对每一类目标，使用一个线性SVM二类分类器进行判别。输入为深度网络输出的4096维特征，输出是否属于此类。 
由于负样本很多，使用hard negative mining方法。 样本判别方法为：

- 正样本：本类的真实值
- 负样本：考察每一个候选框，如果和本类所有标定框的重叠都小于0.3，认定其为负样本。

##### regression

1. 目标检测问题的识别精度为：**mAP**（mean average precision）

- **precision**：对于某张图片计算object C在图片上的查准率
  - 某图片上C识别正确的个数/ 某图片C的总个数
  - $precision_{c} = N(TruePositives)_{c} / N(TotalObjects)_{c}$
- **average precision**：对于object C在多张图片上的查准率
  - 每张图片的precisionc的和 / 含有object C的图片的数目
  - $averagePrecision_{c} = sum(precision_{c}) / N(TotalImages)_{c}$
- **mean average precision**:对整个数据集的多个object的平均查准率
  - 每个objectc的average precision / 总的object数目
  - $meanAveragePrecision = sum(averagePrecision_{c}) / N(classcs)$



2. 目标检测定位精度：**IoU**（intersection over union ）

- 检测结果（detection result）和真实值（ground truth）的交集 / 其并集
- IoU = (GT∩DR) / (GT∪DR) 



3. 机器学习中的评价指标：**accuracy、precision、recall **

- TP（true positive）正样本中预测为正样本        正确预测为正样本
- FP（false positive）负样本中预测为正样本       错误预测为正样本
- FN（false negative）正样本中预测为负样本     错误预测为负样本
- TN（true negative）负样本中预测为负样本      正确预测为负样本 

accuracy：预测正确的样本数                                       accuracy = (TP + TN) / (TP + FP + FN + TN) 

precision：预测为正的样本中，正确预测的比例        precision = TP / (TP + FP) 

recall：正样本中，正确预测的比例                              recall = TP / (TP + FN)   

F1: precision和recall的综合指标                                  F1 = 2×(precision / (precision + recall)) 

4. 回归器

在R-CNN中，对每一类目标，使用一个线性脊回归器进行精修。正则项$\lambda=10000$。  输入为深度网络pool5层的4096维特征，输出为xy方向的缩放和平移。  判定为本类的候选框中，和真值重叠面积大于0.6的候选框为正样本。



### 优缺点分析

#### 优点

- 首次使用CNN进行特征提取
- 使用bounding box regression进行目标包围框的修改 

#### 缺点

- selective search算法较为耗时，每帧图像需要2s处理时间
- CNN前向传播较为耗时，对每一个RoI（region of interest）都需要输入AlexNet提取特征
- 三个步骤分别训练，消耗存储空间 









