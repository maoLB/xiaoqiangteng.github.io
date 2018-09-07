---
title: 论文阅读-PDR
date: 2018-07-13 10:06:04
mathjax: true
categories:
- 学术
- PDR
tags:
- 学术
- PDR
copyright:
---

# 论文一、IONet: Learning to Cure the Curse of Drift in Inertial Odometry

## Motivation

现有的基于惯性传感器的位置估计方法

![](http://p55se4hrx.bkt.clouddn.com/images/science/PDR/IONnet_01.jpeg)

**不足**：

1. 累计误差无界；
2. SINS的累计误差为指数级增加，PDR的累计误差为线性增加；
3. PDR对于无法检测到步子的场景失效。

现有工作包括：

（1） 接连惯性导航方法（Strapdown Inertial Navigation System，SINS）

方法主要思想：双重积分来估计位置和速度。

不足：由于MEMS传感器的成本有限，其测量精度有限，导致位置和速度估计误差呈指数级增长。

（2）PDR （Pedestrian Dead Reckoning）

方法主要思想：计步、步长估计、方向估计

不足：累计误差呈线性增长，对于无法检测步子的场景失效。

**本文贡献**：

（1）建模惯性追踪问题为序列学习问题；

（2）本文提出了基于DNN的学习框架。

## 背景知识

(1) 方位姿态更新

公式（1）：$$C_{b}^{n}(t)=C_{b}^{n}(t-1)\*\Omega(t),$$

公式（2）：$$\sigma=w(t)dt,$$

公式（3）：$$\Omega(t)=C\_{b\_{t}}^{b\_{t-1}}=I+\frac{sin(\sigma)}{\sigma}[\sigma X]+\frac{1-cos(\sigma)}{\sigma^{2}}[\sigma X]^2,$$

(2) 速度更新

$$v(t)=v(t-1)+((C_{b}^{n}(t-1))\*a(t)-g\_{n})dt,$$

(3) 位置更新

$$L(t)=L(t-1)+v(t-1)dt,$$

其中，$C_{b}^{n}$为本体坐标系到导航坐标系的旋转矩阵，$\Omega(t)$为时间间隔t内的旋转变化矩阵，a和w分别表示加速度计读数和陀螺仪读数，v和L分别表示世界坐标系下的速度和位置，g是重力值。

## 问题分析

在SINS和PDR中，惯性传感器数据并不是独立的，系统状态都是有迁前一时刻的系统状态和当前时刻的惯性传感器观测值计算得到的。因此，在给定时间窗口内，惯性传感器数据不是独立的。本文假设一个伪独立条件的存在，即给定时间窗口内，**导航状态的变化是独立的**。因此，本文基于此，提出了序列学习的模型来估计追踪状态。

不可观测到的系统状态包括方向$C_{b}^{n}$，速度v和位置L。在传统模型中，计算公式定义如下：

$$[C_{b}^{n}\ v\ L]=f([C_{b}^{n}\ v\ L]\_{t-1}, [a\ w]\_{t}).$$

(1) 位移L

为了将当前窗口的位移与前一时间窗口的位移分割开来，本文定义了位移变化量$\Delta L$，其是在时间窗口内是独立的，我们得到：

$$\Delta L=\int_{t=0}^{n-1}v(t)dt.$$

继续划分，位移主要由初始速度和加速度计读数计算得到，即：

$$\Delta L=nv(0)dt+[(n-1)s_1+(n-2)s_2+\dots+s_{n-1}]dt^2,$$

其中，

$$s(t)=C_{b}^{n}(t-1)a(t)-g.$$

其中，$s(t)$表示的是世界坐标系下加速度计读数的变化值。那么$\Delta L$就等于第n时刻初始速度对时间的积分加上后续时刻对加速度计的积分。又由公式（1）得：

$$\Delta L=nv(0)dt+[(n-1)C_b^n(0)\*a_1+(n-2)C_b^n(0)\Omega(1)\*a_2+\dots+C_b^n(0)\prod_{i=1}^{n-2}\*a\_{n-1}]dt^2-\frac{n(n-1)}{2}gdt^2,$$

进一步得到：

$$\Delta L=nv(0)dt+C_b^n(0)Tdt^2-\frac{n(n-1)}{2}gdt^2,$$

其中，

$$T=(n-1)a_1+(n-2)\Omega(1)a_2+\dots+\prod_{i=1}^{n-2}\Omega(i)a\_{n-1},$$

由于本文关注的是水平面上人员的位置追踪问题，因此，竖直轴的变化量假设为0，那么行走位移变化量可表示为：

$$\Delta l=||\Delta L||_2=||nv^b(0)dt+Tdt^2-\frac{n(n-1)}{2}g_0^bdt^2||_2.$$

因此，水平移动距离的变化量可由初始速度、重力值、加速度计读数和加速度读数来计算得到，即：

$$\Delta l=f(v^b(0),g_0^b,a_{1:n},w_{1:n}).$$

此外，方向变化为$\phi$，我们得到：

$$(\Delta l, \Delta \phi)=f_{\theta}(v^b(0),g_{0}^{b},\hat{a}\_{1:n},\hat{w}\_{1:n}),$$

其中，$\hat{a}$和$\hat{w}$分别为IMU的加速度计读数和陀螺仪读数的原始值。

最后，为了得到全局的位置信息，我们得到如下公式：

$$x_n=x_0+\Delta l cos(\phi_0+\Delta \phi)$$

$$y_n=y_0+\Delta l sin(\phi_0+\Delta \phi),$$

其中，$(x_0,y_0)$为初始位置。

## 深度神经网络框架

系统框架：

![](http://p55se4hrx.bkt.clouddn.com/images/science/PDR/IONet_02.jpeg)

学习模型：

$$(a, w)_{200\*6}\underrightarrow{f\_{\theta}}(\Delta l, \Delta \phi)\_{1\*2},$$

其中，时间窗口为200帧，即2秒。

本文采用LSTM结构来训练。损失函数定义为：

$$loss=\sum||\Delta \hat{l}-\Delta l||_{2}^{2}+k||\Delta \hat{\phi}-\Delta \phi||\_{2}^{2}.$$

## 实验

### 同一设备不同姿态

![](http://p55se4hrx.bkt.clouddn.com/images/science/PDR/IONet_03.jpeg)

### 不同设备

![](http://p55se4hrx.bkt.clouddn.com/images/science/PDR/IONet_04.jpeg)

### 行走轨迹

![](http://p55se4hrx.bkt.clouddn.com/images/science/PDR/IONet_05.jpeg)

### 小车行走轨迹

![](http://p55se4hrx.bkt.clouddn.com/images/science/PDR/IONet_06.jpeg)




















