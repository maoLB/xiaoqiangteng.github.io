---
title: VIO教程
date: 2018-06-14 11:35:25
mathjax: true
categories:
- 编程
- VIO
tags:
- 编程
- VIO
copyright:
---

# VIO背景

* * *
- - -

# 基础知识

## IMU建模

### 误差

基本误差：零偏误差、刻度误差。但实际上环境温度会对这两种误差均造成大大小小的影响（ADC温漂）。

（1）所以首先我们需要对传感器进行恒温处理，通常是在IMU的PCB板子上加一两个恒温电阻，利用传感器本身的温度测量功能进行加热闭环控制，我一般把这个目标温度设定在50-60度之间。

（2）实现恒温后，我们再开始校准传感器的两种误差：

陀螺仪。零偏误差可以通过在传感器静止时采集输出数据并取其平均值，刻度误差需要通过高精度转台来对每个轴进行校准；

加速度计。首先建立一个基本的误差方程，g为刻度误差，o为零偏误差，C为重力加速度，u为误差：

$$(g_xa_x+o_x)^2+(g_ya_y+o_y)^2+(g_za_z+o_z)^2-C^2=u.$$

这实际上就是一个椭球的方程，为了求解这个方程，我们需要采集多组传感器数据。这些数据尽量在球面上分布均匀（如常用的六面校准），然后最小二乘法求方程的最优解（使u最小），得到传感器的校准参数。

电子罗盘传感器也可以用这个方法来校准。



## 图像特征提取算法

### 斑点检测 - LoG与DoH

### 斑点检测 - SIFT

### 斑点检测 - SURF

### 角点检测 - Harris角点

### 角点检测 - FAST角点

在实时的视频流处理中，需要对每一帧特征提取，对算法处理速度上有很高的要求，传统的SIFT,Harris等特征点提取很难满足。由此提出Fast（Features from Accelerated Segment Test），由于不涉及尺度，梯度，等复杂运算，Fast检测器速度非常快。它使用一定邻域内像元的灰度值与中心点比较大小去判断是否为一个角点。但它的缺点是不具有方向性,尺度不变性。

Fast角点提取步骤（以Fast-12-16为例）：

1.以固定半径为圆的边上取16个像素点（图中白色框出的位置），与中心点像素值Ip做差。

2.若边上存在连续的12（N>12,若为Fast-9,只需要N>9）个点满足  ( I(x)-I(p) )>threshold 或者 ( I(x)-I(p) ) < -threshold。(其中I(x)表示边上的像素值，I(p)为中心点像素值，threshold为设定的阈值。)则此点作为一个候选角点。如图上的虚线连接的位置。通常为了加速计算，直接比较1,5,9,13位置的差值，超过三个即视为一个候选点（存在连续的12个像元的必要条件），否则直接排除。

3.非极大值抑制，排除不稳定角点。

### 二进制字符串特征描述子 - BRIEF算法

### 二进制字符串特征描述子 - BRISK算法

### ORB算法

### FREAK算法

* * *
- - -

# 基于滤波的VIO方法

## 基于松耦合的方法

## 基于紧耦合的方法

### 论文1. MSCKF: High-precision, consistent EKF-based visual-inertial odometry

> the multi-state-constraint Kalman filter (MSCKF)



* * *
- - -

# 基于优化的VIO方法

## 基于松耦合的方法

## 基于紧耦合的方法

### VINS文献解读

#### 测量预处理

##### 视觉预处理前端

（1）对每张新图像，使用KLT稀疏光流算法跟踪已有特征。

（2）同时检测新的角点特征以维持每张图像100-300特征。其中，新的角点特征利用Good Features to track特征点检测算法，参考文献：《Good Features to track》，[opencv](http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_feature2d/py_shi_tomasi/py_shi_tomasi.html#shi-tomasi)，详细讲解参考：[Good Features to track特征点检测原理与opencv（python）实现](https://blog.csdn.net/u010103202/article/details/73331440)。

（3）检测器通过设置相邻特征最小间隔强制特征均匀分布，outlier去除后，特征投影在单位球面上（unit sphere）。Outlier通过RANSAC基础矩阵测试去除。

（4）关键帧的选取原则有两个：平均视差和跟踪的特征数。当跟踪的特征数小于某个门限或者跟踪特征的平均视差超过某个门限时，插入关键帧。请记住，除了平移，旋转也会导致视差，但是纯旋转时特征无法三角定位，我们在计算视差时用IMU propagation结果补偿旋转。

* * *
- - -

# 参考

- https://www.zhihu.com/question/53571648
- http://baijiahao.baidu.com/s?id=1573826383552981&wfr=spider&for=pc
- https://blog.csdn.net/luoshixian099/article/details/48294967
- https://www.jianshu.com/p/2759593bc92b
- https://www.jianshu.com/p/387225a1aa60