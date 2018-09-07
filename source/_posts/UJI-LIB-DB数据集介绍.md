---
title: UJI_LIB_DB数据集介绍
date: 2018-06-07 16:57:50
mathjax: true
categories:
- 编程
- 数据集
tags:
- 编程
- 数据集
copyright:
---

# UJI_LIB_DB介绍

## 论文出处

> Mendoza-Silva, Germán Martín, Richter, Philipp, Torres-Sospedra, Joaquín, Lohan, Elena Simona, & Huerta, Joaquín. (2017). Long-Term Wi-Fi fingerprinting dataset and supporting material (Version V1) [Data set]. Zenodo. http://doi.org/10.5281/zenodo.1066041

## 详细介绍

```bash
时间跨度：    15个月
采集点数：    63504
采集地点：    大学图书馆（3楼和5楼）
AP放置位置：  天花板
AP放置高度：  2.65米
具体位置：    详见数据集
AP数量：     448
采集人员数量：1名
采集方式：   右手放至胸前，每个采样点采集6个数据
采集设备：   Samsung Galaxy S3
采集软件：   自研
```

数据库格式定义：

$$DB=\{D_{(m,k,n)}\}$$

其中，$D_{(m,k,n)}$表示在第m月份捕获的第k类型的第n个数据集。每个数据及定义四个集合：RSS值、位置、时间和标识符集合。RSS值集合定义为：

$$R_{(p\*s)\*a}=\{r_{i,j}\},$$

其中，p表示的是采样点的数量，s表示的是每个采样点采集的数据数量，a表示的是检测到的AP的数量，$r_{i,j}$表示的是第i个指纹第j个AP的RSS值。如果AP没有被检测得到，那么该AP的RSS值为100.

位置集合定义为：

$$L_{p\*s}\*3=\{(x_i,y_i,f_i)\},$$

其中，$(x_i,y_i)$为二维坐标，$f_i$为第i个指纹被采集的楼层号码。

时间集合被定义为：

$$T_{(p\*s)\*1}=\{t_i\},$$

其中，$t_i$是第i个指纹被采集的时间戳。

标识符集合被定义为：

$$ID_{(P\*S)\*1}=\{id_{i}\},$$

其中，$id_{i}$为第i个指纹的标识符。例如：

![](http://p55se4hrx.bkt.clouddn.com/images/programming/datasetsUCI_LIB_DB_id.jpeg)

数据集文件目录：

(“trn” for training, “tst” for test, “rss”, “crd”, “tms” and “ids” for RSS values, positions, times and identifiers sets)

## 采集方法



# 参考
- http://www3.uji.es/~jtorres/databases.html
- https://zenodo.org/record/1066041#.WxjzeVOFPq0
