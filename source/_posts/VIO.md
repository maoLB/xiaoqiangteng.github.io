---
title: VINS-Mobile版源码阅读
date: 2018-09-08 9:56:20
categories:
- 编程
- IOS
tags:
- 编程
- IOS
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# VIO基础知识

## IMU是什么

IMU中文名叫惯性测量单元，英文名：Inertial measurement unit，简称 IMU。简单理解就是这个东东可以测量两个东西，加速度 a 是沿三个轴\\(a\_{x}\\),\\(a\_{y}\\),\\(a\_{z}\\), 方向的线加速度，而角速度 w 就是这三个方向的角速度 \\(w\_{x}\\),\\(w\_{y}\\),\\(w\_{z}\\), 还有就是IMU的频率比较高一般都在100HZ以上。在IMU内部，除了通常的白噪声，还有个特别的量零偏bias，在这是传感器内部机械、温度等各种物理因素产生的传感器内部误差的综合参数。IMU的加速度计和陀螺仪的每个轴都用彼此相互独立的参数建模，一个角速度测量值和真值之间的连续域上的关系可以写作：

$$\tilde{w}=w+b\_{w}+n\_{w}.$$

在纯视觉的slam或者vo中，由于图像的运动模糊、遮挡、快速运动、纯旋转、尺度不确定性的一系列问题，导致仅靠一个摄像头很难完成我们实际场景的应用需求，而IMU直接可以得到运动主体自身的角速度、加速度的测量数据，从而对运动有一个约束，或者说与视觉形成互补，可实现快速运动的定位和主体纯旋转的处理，从而进一步提高slam/vio的可靠性。

VIO中IMU与图像之间的时间序列关系如下图所示：

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU-Pre_01.jpg)

图1 IMU与视觉在时间轴上的信息

如图1所示。在时间轴上，IMU通常以较快的速率采集角速度和加速度的信息，而视觉则是以较慢的频率采集图像。VIO的器件同步，保证了每个时刻采集的数据都是同步的。原理上，我们可以给出每个时刻的位姿估计，然而，现有视觉SLAM，多数是基于关键帧+BA的处理形式。于是，一个重要的问题是，能否将两个视觉关键帧当中的IMU数据，整合在一起，约束它们之间的运动？如果可以的话，又如何来约束？预积分的目的，就在于处理这里的运动关系。为了说清楚这件事，需要介绍一些背景知识。

(1) 坐标系和动力学

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_02.png)

图2 VIO器件的坐标系

在图2中，当IMU在世界中运动时，我们有世界坐标系（W），IMU坐标系（B）和像极坐标系（C）。对于IMU，通常估计世界到IMU坐标系的位姿为\\(R_{WB}\\)和\\(P\_{WB}\\)，其中为了与时间进行区分，这里的平移采用P来表示。图2右侧给出了动力学方程，即位移的微分为速度、速度的微分为加速度、旋转的微分为角速度。

（2）IMU测量数据表示

IMU可测量IMU系（B）的角速度和加速度，测量信号受噪声和零偏的影响，我们得到：

B系下的角速度：

$$\_B\tilde{w}_{WB}(t)=\_Bw\_{WB}(t)+b^{g}(t)+\eta^{g}(t),$$

加速度，考虑世界坐标系下的重力：

$$\_B\tilde{a}(t)=R\_{WB}^{T}(t)(\_Wa(t)-\_Wg)+b^{a}(t)+\eta^{a}(t),$$

IMU的Bias方程：

$$\dot{b}^{g}(t)=\eta^{bg},$$

$$\dot{b}^{a}(t)=\eta^{ba},$$

其中，\\(\eta^{g}, \eta^{a}, \eta^{bg}, \eta^{ba}\sim N\\)，即服从高斯分布。

为了估计位姿，首先要选择状态变量。在紧耦合（Tightly Coupled）方案中，通常选择位姿、速度、零偏这几个量，作为待估计的状态量，共15维：

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_03.png)

于是，可以想见，在带IMU的bundle adjustment中，我们每一个Pose都是这样一个15维的变量。那么，如何通过IMU数据，定义两个状态量之间的运动约束呢？还是回到动力学。

前面给出了微分形式的动力学，当然我们可以把它写成积分形式的。然后，由于是在离散时刻进行采样，所以得到的是离散时刻的动力学方程。再代入IMU的测量，有：

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_04.png)

个人整理：

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_04_1.png)

于是，差分方程给出了两个连续的视觉帧之间的IMU约束。进一步，如果把两个关键帧之间的多个视觉帧积分起来，就形成了预积分：

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_05.png)

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_06.png)

个人整理：

![](http://p55se4hrx.bkt.clouddn.com/images/science/VIO/IMU_Pre_06_1.png)


## IMU 预积分

IMU的数据频率一般远高于视觉，在视觉两帧k，k+1之间通常会有>10组IMU数据。IMU的数据通过积分，可以获取当前位姿（p位置，q四元数表达的姿态）、瞬时速度等参数。

在VIO中，如果参考世界坐标系对IMU进行积分，积分项中包含相对于世界坐标系的瞬时旋转矩阵，这样有几个问题：

1. 相对世界坐标系的旋转矩阵有drift，如果一直以其为基准进行积分，必然造成积分误差累积；

2. 在进行优化位姿调整时（通常是调整视觉KeyFrame的pose），相对于世界坐标系的pose会变化，因而优化后的瞬时旋转矩阵和积分时不同，那么积分自然也就存在问题；

3. 一般这个旋转矩阵不知道。。。

因此，一般的预积分的参考坐标系为k帧的IMU参考系，这样可以解决以上问题：

1. 相对k帧的IMU进行积分，不会有累积误差；

2. 即使后面调整了位姿，相对位置不变，因此预积分不存在问题；

3. 这个旋转矩阵为单位矩阵E，后面每出现一个IMU数据，都可以用任何一种数值积分的方法计算；同时可以将重力加速度提取到积分号外面不参加积分，相当于在重力参考系中积分，计算量也会减少。

## 参考

- https://www.cnblogs.com/shang-slam/p/7080113.html
- https://zhuanlan.zhihu.com/p/38009126
- [预积分 | Momenta Paper Reading 七期回顾](https://zhuanlan.zhihu.com/p/26243851)
- [如何理解IMU以及其预积分](https://zhuanlan.zhihu.com/p/38009126)


## 论文一：Visual-Inertial-Aided Navigation for High-Dynamic Motion in Built Environments Without Initial Conditions

Todd Lupton and Salah Sukkarieh

The University of Sydney

### Motivation

1. 位置追踪具有广泛的应用，诸如安全救援、自动驾驶等。

2. 当前VO或SLAM方法均存在一些局限性，如VO没有尺度信息。此外，IMU能够提供尺度信息，但难点在于初始化过程。

### 本文主要想法及贡献点

> This core idea is expanded in this paper with integration into a graphical filter that allows automatic inertial initialization and map management to produce a robust visual-inertial navigation system

本文的核心思想是扩展到一个图形滤波器中，它允许自动惯性初始化和地图管理来产生一个鲁棒的视觉惯性导航系统

> The main contribution of this paper is that no explicit initialization stage or large uncertainty priors are required, and the initial conditions are automatically recovered in a linear manner.

贡献点是：无需初始化阶段或大的不确定性先验，并且初始条件自动地以线性方式恢复。






























