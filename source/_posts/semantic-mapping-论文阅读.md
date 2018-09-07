---
title: semantic_mapping_论文阅读
date: 2018-03-26 15:36:34
categories:
- 学术
- 论文阅读
- Semantic Mapping
tags:
- 学术
- 论文阅读
copyright:
---

# Semantic Mapping 文献综述

> 1，CNN-SLAM为今年CVPR的文章，是比较完整的pipeline，将LSD-SLAM里的深度估计和图像匹配都替换成基于CNN的方法，取得了更为robust的结果，并可以融合语义信息。见[ProjectCNNSLAM](http://campar.in.tum.de/Chair/ProjectCNNSLAM).类似的工作还有UnDeepVO: Monocular Visual Odometry through Unsupervised Deep Learning。 问题在于准确度非常低。做过benchmark，基于单帧彩色照片进行距离信息预测，在室内每个像素的平均误差约50cm，在室外平均误差则高达７米以上。一篇投ICRA的工作结合少量的距离信息和彩色信息进行距离图像预测，效果比单纯用彩色照片准确得多且鲁棒性强。这个方法可以帮助传统SLAM从稀疏点云快速生成密集的点云，也可以用在激光雷达的超分辨率上。代码已开源[sparse-to-dense](https://github.com/fangchangma/sparse-to-dense),论文：Sparse-to-Dense: Depth Prediction from Sparse Depth Samples and a Single Image </br ></br >
> 2. VINet是AAAI2017的文章，利用CNN和RNN构建了一个VIO，即输入image和IMU信息，直接输出估计的pose。 </br ></br >
> 3. Unsupervised learning of depth and ego-motion from video是Google CVPR 2017的oral文章，利用CNN学习一个无监督的深度估计和pose估计网络，代码见[SfMLearner](https://github.com/tinghuiz/SfMLearner.git).SfM-Net利用监督学习也干了类似的工作。 </br ></br >
> 4. 重定位PoseNet和Delving deeper into convolutional neural networks for camera relocalization </br >
> 5. 语义地图 Semi-Dense 3D Semantic Mapping from Monocular SLAM </br ></br >
> 6. 哪怕在传统基于特征点的SLAM已经能做到非常稳定高效的这个情况下，适合结合deep learning的科研方向还是有很多的。这其中包括 </br >
>> - 提高特征点稳定性（减少outlier）和自动提取不同层级的特征点（点、线、面、物体）， </br >
>> - 快速生成密集的地图（而非稀疏的三维点云） </br >
>> - 结合语义信息和图像分割 </br >
>> - 生成动态地图（可以实时更新、表达动态物体） </br >
>> - 降低SLAM调参的难度 </br >

> 7.跳出SLAM，说点题外话，利用深度强化学习来进行端对端的机器人导航，已经有了不错的结果。人类在环境中导航，不也是直接输入image，输出action吗？有兴趣的可以看看这两篇文章：[Cognitive Mapping and Planning for Visual Navigation](https://arxiv.org/abs/1702.03920)和[Target-driven Visual Navigation in Indoor Scenes using Deep Reinforcement Learning](https://arxiv.org/abs/1609.05143)。navigation 是更适合 DL 的一个场景。人在移动的时候，并不会建立精确的环境地图，无法具体说出障碍物距离自己多少厘米。所以，我一直有一个直觉：「navigation 应该不需要精确地图信息与定位信息」，而 DL 似乎有可能实现这一全新的方法。 navigation 是更适合 DL 的一个场景。人在移动的时候，并不会建立精确的环境地图，无法具体说出障碍物距离自己多少厘米。所以，一直有一个直觉：「navigation 应该不需要精确地图信息与定位信息」，而 DL 似乎有可能实现这一全新的方法。但是，目前所有这些工作都存在一个问题：只是训练出一个 local planner，无法实现全局的路径规划。 </br ></br >
> 8. 语义分割和SLAM的结合还很粗糙.最简单的方式，就是跑一个pixel-wise的图像语义分割，再跑一个dense或者semi-dense的SLAM，把前者的结果map到后者的地图上去，每个像素（或者surfel）上做recursive Bayesian update，其实也就是概率累乘。参见Andrew Davison组的SemanticFusion (ICRA'17)，代码已开源。国内学者也有类似的工作，用LSD-SLAM + DeepLab v2，这个是单目的（SemanticFusion是RGB-D）,即[Semi-Dense 3D Semantic Mapping from Monocular SLAM](https://arxiv.org/abs/1611.04144)。这种结合方式，按某些学者的意见都不能称为semantic SLAM，只能叫semantic mapping，因为localization部分跟semantics没关系嘛。 </br ></br >
> 9. 真正的semantic SLAM，语义信息是要能够帮助定位的，比如这篇：[Probabilistic Data Association for Semantic SLAM](https://pdfs.semanticscholar.org/ef4c/ffbbca79df1c1ca7891345f898812289b6cb.pdf)(ICRA'17)。用object detection的结果作为SLAM前端的输入，跟ORB之类的特征互补提高定位鲁棒性。优点很明显，这下SLAM不会因为你把床收拾了一下就啥都不认识了（视觉特征都变了，但床还是床）。难点是detection结果的data association最好能跟定位联合优化，但前者是个离散问题。这篇文章用EM算法，E步考虑所有可能的association，比较粗暴，但识别物体较少的时候还不错（论文实验里只识别椅子）。上面这篇文章没有语义分割。。。map里只有稀稀拉拉的几个物体（位置和类别）。 </br ></br >
> 10. 另外，SLAM也能提升语义理解水平。前面提到的SemanticFusion和类似的工作里，融合了多个视角语义理解结果的3D地图，其中的语义标签准确率高于单帧图像得到的结果，这很容易理解。另外，通过在3D空间引入一些先验信息，比如用CRF对地图做一下diffusion，能进一步提升准确率。但CRF毕竟还是简单粗暴，如果设计更精细的滤波算法，尤其是能从真实数据中学习一些先验的话，应该效果还会更好。这方面的工作还没有。 </br ></br >
> 11. 再提一个，融合优化之后的结果如果反馈给图像语义理解算法做一下fine-tuning，那就是self-supervised learning了。这方面的工作也还没有。 </br ></br >
> 12. 接下来是语义地图怎么用的问题。对上层应用有价值的语义地图，应该包含一个个物体及其模型，而不仅是一堆标记了类别的voxel。一个比较好的例子来自IROS'17：[Meaningful Maps With Object-Oriented Semantic Mapping](https://arxiv.org/abs/1609.07849)不过这篇文章里的语义信息来自SSD和非神经网络的分割，还没有用端到端的语义分割网络。 </br ></br >
> 13. 另外，针对动态场景，怎样处理物体移位，怎样区别长效地图和短效地图，怎么“脑补”同类物体，这里面一堆问题可以研究。更别说地图构建出来之后如何做体现空间智能的自然语言交互和任务规划，如何reasoning了，这方面研究目前连影子都没有。 </br ></br >
> 14. 一篇很好的综述，里面也有很多关于语义SLAM的介绍：[Past, Present, and Future of Simultaneous Localization And Mapping: Towards the Robust-Perception Age](https://arxiv.org/abs/1606.05830) </br ></br >
> 15. 语义slam开源：</br >
> - [DA-RNN_Semantic Mapping with Data Associated](https://github.com/yuxng/DA-RNN)</br >
> - [SemanticFusion](https://bitbucket.org/dysonroboticslab/semanticfusion)</br >
> - [Pop-up SLAM: Semantic Monocular Plane SLAM](https://github.com/shichaoy/pop_up_image).场景理解用于改善状态估计，尤其是在低纹理区域，是目前极少的开源语义SLAM方案之一</br >


参考：
1. [当前深度学习和slam结合有哪些比较好的论文，有没有一些开源的代码?](https://www.zhihu.com/question/66006923/answer/241333356)
2. [当前语义分割帮助视觉SLAM提高定位准确度，建立语义地图的研究现状如何？](https://www.zhihu.com/question/264578623/answer/283163990)

## 研读论文一：CNN-SLAM: Real-time dense monocular SLAM with learned depth prediction

> Tateno, K., Tombari, F., Laina, I., & Navab, N. (2017). CNN-SLAM: Real-time dense monocular SLAM with learned depth prediction. arXiv preprint arXiv:1704.03489.
> CVPR 2017

### Motivation

1. 本文提出了一种基于深度神经网络方法能够从单目图像中预测深度信息，用途可为弹幕图像的重建。尤其是对于纯单目重建是小的区域，如纹理不丰富的区域，效果较好。此外，相比较于单目SLAM，该深度SLAM还可提供scale。此外，该方法还融合了语义标记，来重建语义信息的场景。
2. 传统的基于深度摄像头的SLAM有如下缺点：
- 有效工作距离较短
- 太阳光的影响等
- 普适性
3. 双目摄像头缺点：特征丰富程度敏感
4. 鉴于卷积神经网络（CNN）深度预测的最新进展，本文研究了深度神经网络的预测深度图，可以部署用于精确和密集的单目重建。我们提出了一种方法，其中CNN预测的稠密深度图与通过直接单目SLAM获得的深度测量自然地融合在一起。我们的融合方案在图像定位中优于单目SLAM方法，例如沿低纹理区域，反之亦然。我们展示了使用深度预测来估计重建的绝对尺度，从而克服了单眼SLAM的主要局限性之一。最后，我们提出一个框架，从单个帧获得的语义标签有效地融合了密集的SLAM，从单个视图产生语义相干的场景重构。两个基准数据集的评估结果显示了我们的方法的鲁棒性和准确性
