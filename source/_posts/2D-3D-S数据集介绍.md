---
title: 2D-3D-S数据集介绍
date: 2018-09-14 08:25:03
categories:
- 编程
- 数据集
tags:
- 编程
- 数据集
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# 引言

数据集详情地址：http://3Dsemantics.stanford.edu/

该数据集构建的主要目标是为场景理解、深度估计、法线估计、实体检测、分割和场景重建提供研究数据集，推动该领域的发展。

# 相关数据集介绍

现有的基于RGB-D数据集包括NYU Depth v2、SUN RGBD和SceneNN。然而，这些数据集对于处理特定的任务和2.5D的场景有限。

# 采集和处理

## 3D模式

3D模式主要包含点云和mesh。文章利用Matterport Camera得到3D textured Mesh model，然后利用采样方法得到3D点运数据。此外，文章将点运数据与颜色进行对应来生成colored点云数据。

语义标记：文章对点云数据中的每个Mesh和Voxel进行语义信息的标记。语义主要包括ceiling, floor, wall, beam, column, window, door, table, chair, sofa, bookcase, board and clutter。此外，语义信息被投影到RGB图像中。进一步地，文章划分数据为如下室内场景：office, conference room, hallway, auditorium, open space, lobby, lounge, pantry, copy room, storage and WC。每个对象被存储在文件名为“class instanceNum roomType roomNum areaNum”中。

## 2D模态

RGB图像：采集朝向：yaw角约[-180, 180], pitch角约为以0为均值15度为方差的高斯分布，roll角一直是0度。FOV角为以75度为均值和-30度为标准差的半高斯分布。为了过滤掉白墙等图像，文章基于语义信息商来过滤掉近70%的图像。

语义信息商的计算方法如下：首先，对于每一幅图像对13类别的语义信息进行像素统计。再利用像素分布计算香农商。最后，将低于15%的图像过滤掉。

此外，我们保留60%的图像。剩下的25%的图像我们通过半高斯分布采样方法来筛选。如此，文章保证了图像的多样性。

