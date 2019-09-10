---
title: C3D模型实验
date: 2019-03-19 16:04:59
tags:
- 计算机视觉
- 毕设项目
- 动作识别
categories: Work
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.png
---

# C3D模型实验

## 实验环境

实验室服务器，将pycharm远程连接到服务器，选择python3节点，下载UCF101数据集，上传服务器，解压，其中解压rar压缩包需要下载RAR for Linux，到[官网](http://www.rarsoft.com/download.htm)下载，然后解压。

```shell
tar -xzvf rarlinux-5.5.0.tar.gz
```

将其拷贝到/usr/local/目录下

```shell
cp -r rar /usr/local/
```

cd到rar文件夹下，make即安装完成。接下来解压UCF数据集。

```shell
unrar e UCF101.rar
```

### 数据预处理

因为数据集全是视频，因此需要抽帧，运行convert_video_to_images.sh文件，首先需要安装dos2unix

```
sudo apt-get install dos2unix -y
dos2unix convert_video_to_images.sh
sudo chmod u+x convert_video_to_images.sh
sudo ./convert_video_to_images.sh UCF101/ 5
```

视频按照5fps提取图片。

在运行convert_images_to_list.sh生成训练集和测试集的list，sh处理方法和上一步相同，需要先安装jot

```
sudo apt install athena-jot
dos2unix convert_images_to_list.sh
sudo chmod u+x convert_images_to_list.sh
sudo ./convert_images_to_list.sh UCF101/ 4
```

表示按照4:1划分训练集和测试集。

### 训练数据

使用Python3训练数据需要注意Python2到Python3中发生的变化，更改掉冲突之后就可以运行了，但运行结果过拟合。

运行结果如下：

```
Step 4991: 5.263 sec
Step 4992: 5.513 sec
Step 4993: 4.211 sec
Step 4994: 4.242 sec
Step 4995: 6.805 sec
Step 4996: 5.058 sec
Step 4997: 4.866 sec
Step 4998: 5.036 sec
Step 4999: 4.672 sec
Training Data Eval:
accuracy: 0.95000
Validation Data Eval:
accuracy: 0.65000
train 5000 steps，batch_size：10
```

batch_size表示每个batch的video数。

测试时batch_size设置为1，观察测试时时间约为0.1~0.3spv

